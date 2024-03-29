.. include:: <s5defs.txt>

Plone Conference 2011: Introduction to WSGI
===========================================

:Authors:  Chris McDonough
:Date: 11/3/2011

Intro
-----

- Pylons Project (Pyramid/Pylons-the-web-framework/other libs).

- Agendaless Consulting.

WTF?
------

Q: I keep hearing about this WSGI (Web Services Gateway Interface) thing.
Why should I care?

`A: Using WSGI will allow your application or framework stay relevant longer`

`A: Using WSGI will allow you a better choice of web servers for your
application`

WTF (2)?
--------

Q: I keep hearing about this WSGI thing.  Why should I care?

`A: Using WSGI will let you take advantage of utility code developed as
middleware`

`A: Using WSGI will make transitioning between systems easier`

If You Want to Leave
--------------------

What you should take away from this talk:

`When possible, use WSGI-aware applications and servers`

`Don't use "bare" WSGI (instead use wrappers like WebOb and Werkzeug)`

What is WSGI?
-------------

- A protocol defining how Python web applications should connect to web
  servers.

- Used by Pylons/TurboGears, Pyramid, Django, Zope/Plone/Grok/Bluebream,
  Bottle, Flask/Werkzeug, CherryPy, Google App Engine.

- Basically all major Python web frameworks and platforms (although some are
  more WSGI-focused than others).

The Real World
--------------

- Startup company SurveyMonkey created Pylons-based software in 2010+.
  Developed a lot of functionality as WSGI middleware.

- Today, switching line of business applications to Pyramid.

  `Their deployment strategy remains the same (even after changing web
  frameworks).`

  `They don't need to change a lot of the code they've already developed as
  WSGI middleware.`

The Real World (2)
------------------

- Zope 2.13 ships with WSGI.

- This means Plone can be served via WSGI servers "for free".

- Lends further life to an older framework.

Back in Time
------------

- We know what WSGI can do for us today.

- Let's go back in time a little, to see its roots.

CGI
---

- Common Gateway Interface.

- CGI was awesome. Simple.

- Not a framework, just a protocol.

- Language agnostic.

CGI Sucks
---------

- Every request invoked a script (process).

- Response generated by a script which printed to stdout.

- Very slow.

CGI Program
-----------

.. sourcecode:: python

   # myscript.cgi (Python implementation)
   qs = dict(cgi.parse_qsl(
             os.environ.get('QUERY_STRING', ''))
   if qs.get('name'):
       name = qs['name']
   else:
       name = 'World'
   greeting = 'Hello %s!' % name
   print 'Content-Type: text/plain'
   print 'Content-Length: %s' % len(greeting)
   print '\n%s' % greeting

CGI and WSGI
------------

- WSGI is like CGI, except a process isn't (usually) started on each request.

- WSGI reuses CGI environ header meanings like ``QUERY_STRING`` and
  ``REMOTE_ADDR``.

- Like CGI, all WSGI HTTP headers sent with the request are available as
  variables named ``HTTP_{headername}``.  E.g. ``HTTP_HOST_NAME``.

- CGI is just a protocol; so is WSGI.  There's no SuperCGI 1.1, just like
  there's no MegaUltraWSGI 1.4b2.  Both are only castles in the air.

CGI vs. WSGI
------------

- CGI is HTTP serialized as a script invocation, WSGI is HTTP serialized as
  a Python function call.

- In CGI, environ is available as ``os.environ``; in WSGI, environ is not
  ``os.environ``, but is passed as a separate dictionary.

- CGI is language agnostic; WSGI is a Python thing.

- In WSGI, extra vals are available as ``wsgi.*`` e.g. ``wsgi.scheme`` and
  ``wsgi.input``.  Extensions of CGI specification.

- CGI is slow; WSGI is fast.

WSGI Application API
--------------------

A WSGI application is a callable with the following signature:

.. sourcecode:: python

    def application(environ, start_response)

A WSGI application must call ``start_response``, then return a sequence of
bytes.

Simplest App
------------

.. sourcecode:: python

   def hello_world(environ, start_response):
       start_response('200 OK', 
                      [('Content-Length', '12'),
                       ('Content-Type', 'text/plain')]
       return ['Hello world!']

Environ
-------

.. sourcecode:: python

   def hello_world(environ, start_response):
       start_response('200 OK', 
                      [('Content-Length', '12'),
                       ('Content-Type', 'text/plain')]
       return ['Hello world!']

``environ`` is a dictionary which contains everything in ``os.environ``
plus variables which contain data about the HTTP request.

Start Response
--------------

.. sourcecode:: python

   def hello_world(environ, start_response):
       start_response('200 OK', 
                      [('Content-Length', '12'),
                       ('Content-Type', 'text/plain')]
       return ['Hello world!']

``start_response`` is a callable which the application calls to set the
status code and headers for a response, before returning the body of the
response.

Start Response 2
----------------

start_response is a callable which has the following signature:

.. sourcecode:: python

    def start_response(status, response_headers,
                       exc_info=None):
        pass

``start_response`` is widely regarded as being quite weird and will likely
not be present in version 2.0 of the WSGI spec, if and when there ever is
one.

Status
------

.. sourcecode:: python

    def start_response(status, response_headers, 
                       exc_info=None):
        pass

``status`` is a string which contains the numeric HTTP status code of the
response followed by the standard HTTP message for that status code. This is
straight-up HTTP.  A few common statii look like this:

"200 OK", "404 Not Found", "500 Server Error"

Headers
-------

.. sourcecode:: python

    def start_response(status, response_headers, 
                       exc_info=None):
        body = 'hello!'
        response_headers = [
            ("Content-Type", "text/html"),
            ("Content-Length", str(len(body)) ),
            ]
            
        start_response("200 OK", response_headers)
        return [body]

Headers (2)
-----------

response_headers is a list of tuples where each tuple is the name of an HTTP
response header and then it's value.

Response headers can be modified by server (or middleware).  At a bare
minimum you should set the ``Content-Type`` header, although the server will
probably substitue ``text/plain`` if you fail to set this.  It is also good
practice to set the ``Content-Length`` header as this allows a client to make
another request on the same socket connection and provide the user with
download progress information.

exc_info
--------

The ``exc_info`` argument is optional and may be used to communicate
traceback data to downstream components.

If passed, ``exc_info`` must be a tuple of the form returned by
``sys.exc_info()``.

exc_info (2)
------------

If your application catches an exception and generates an HTTP error
response, it is a good idea call ``sys.exc_info`` and pass the result here.
The server or other downstream components can potentially use this to provide
error logging or pretty HTML stack traces.  Not very widely used.

Return Value
-------------

.. sourcecode:: python

   def hello_world(environ, start_response):
       start_response('200 OK', 
                      [('Content-Length', '12'),
                       ('Content-Type', 'text/plain')]
       return ['Hello world!']

The return value for the WSGI application is an iterable object. Each item in
the iterable is a chunk of the HTTP response body as a byte string.

wsgi.input
----------

- GET parameters are available from the ``QUERY_STRING`` enmvironment
  setting.

- POST parameters, however, must be read from the ``wsgi.input`` file handle.

.. sourcecode:: python

   import cgi
   form = cgi.FieldStorage(fp=environ['wsgi.input'])

Generators
----------

You can use a generator for the response body.

.. sourcecode:: python

    BLOCK_SIZE = 8192

    def send_file(file_path):
        with open(file_path) as f:
            block = f.read(BLOCK_SIZE)
            while block:
                yield block
                block = f.read(BLOCK_SIZE)

Generators (2)
--------------

The part of our WSGI application that uses the generator might look like
this:

.. sourcecode:: python

   size = os.path.getsize(file_path)
   headers = [
       ("Content-Type", mimetype),
       ("Content-Length", str(size)),
   ]
   start_response("200 OK", headers)
   return send_file(file_path)

Run It
------

There are several server components out there that are able to run WSGI apps,
but for simple testing purposes we can just use the reference implementation
included in Python's standard library.

.. sourcecode:: python

   if __name__ == '__main__':
       from wsgiref.simple_server import make_server
       server = make_server('localhost', 
                            8080, application)
       server.serve_forever()

Middleware
----------

Simplest possible middleware:

.. sourcecode:: python

    class DoNothingMiddleware:
        def __init__(self, app):
            self.app = app

        def __call__(self, environ, start_response):
            return self.app(environ, start_response)

This middleware does nothing.

Middleware (2)
--------------

- ``repoze.who``: authentication middleware (endware).

- ``Deliverance``: HTML transformation middleware.

- ``WebError``: debugging middleware.

- ``repoze.tm``: transaction management middleware.

- ``repoze.profile``: profiling middleware.

- ``Beaker``: sessioning middleware.

Server Choices
--------------

- ``paste.httpserver``

- ``mod_wsgi``

- ``CherryPy`` server.

- ``wsgiref``

Paste
-----

- Set of tools developed by Ian Bicking which makes it slightly easier to
  compose WSGI applications, middleware, and servers together.

- Currently unmaintained.

PasteDeploy
-----------

- Config file format.

- App

.. sourcecode:: python

   [app:myapp]
   use = egg:starter

   pyramid.reload_templates = true
   pyramid.includes = pyramid_debugtoolbar

PasteDeploy (2)
----------------

- Pipeline.

.. sourcecode:: python

   [pipeline:main]
   pipeline = egg:repoze.profile
              myapp

PasteDeploy (3)
---------------

- Server

.. sourcecode:: python

   [server:main]
   use = egg:Paste#http
   host = 0.0.0.0
   port = 6543

PasteScript
-----------

- ``paster`` command for reading PasteDeploy ini files and invoking them,
  plus other miscellanea.

.. sourcecode:: python

   $ bin/paster serve create -t pyramid_starter starter

.. sourcecode:: python

   $ bin/paster serve development.ini

WSGI Sucks
----------

- The WSGI protocol is more complicated than necessary to solve the 99% case
  problem.

- Most Recent PEP: http://www.python.org/dev/peps/pep-3333/

But Nobody Cares
----------------

- Embrace the suck.  WSGI is the worst protocol for connecting apps to
  servers except for every other protocol.

- Nobody cares.  You should never, ever write a low-level WSGI application if
  you value your time (use WebOb or Werkzeug to model requests and
  responses).

Questions
---------

Q?
