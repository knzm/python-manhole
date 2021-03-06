===========================
       python-manhole
===========================

Manhole is a python daemon thread that will accept unix domain socket connections and present the
stacktraces for all threads and an interactive prompt.

Access to the socket is restricted to the application's effective user id or root.

       This is just like Twisted's `manhole <http://twistedmatrix.com/documents/current/api/twisted.manhole.html>`__. 
       It's simpler (no dependencies) and it only runs on Unix domain sockets (in contrast to Twisted's manhole which 
       can run on telnet or ssh).


Usage (you can put this in your django settings, wsgi app file, some module that's always imported early etc)::

    import manhole
    manhole.install() # this will start the daemon thread
    
    # and now you start your app, eg: server.serve_forever()

Now in a shell you can do either of these::

    netcat -U /tmp/manhole-1234
    socat - unix-connect:/tmp/manhole-1234
    socat readline unix-connect:/tmp/manhole-1234

Sample output::

    $ nc -U /tmp/manhole-1234

    Python 2.7.3 (default, Apr 10 2013, 06:20:15)
    [GCC 4.6.3] on linux2
    Type "help", "copyright", "credits" or "license" for more information.
    (InteractiveConsole)
    >>> dir()
    ['__builtins__', 'dump_stacktraces', 'os', 'socket', 'sys', 'traceback']
    >>> print 'foobar'
    foobar


Features
========

* Uses unix domain sockets, only root or same effective user can connect.
* Current implementation runs a daemon thread that waits for connection.
* Lightweight: does not fiddle with your process's singal handlers, settings, file descriptors, etc
* Compatible with apps that fork, reinstalls the Manhole thread after fork - had to monkeypatch os.fork/os.forkpty for this.

What happens when you actually connect to the socket
----------------------------------------------------

1. Credentials are checked (if it's same user or root)
2. sys.__std\*__/sys.std\* are be redirected to the UDS
3. Stacktraces for each thread are written to the UDS
4. REPL is started so you can fiddle with the process


Whishlist
---------

* Be compatible with eventlet/stackless (provide alternative implementation without thread)
* More configurable (chose what sys.__std\*__/sys.std\* to patch on connect time)

Requirements
============

Not sure yet ... maybe Python 2.6 and 2.7. Check Travis:

.. image:: https://secure.travis-ci.org/ionelmc/python-manhole.png
    :alt: Build Status
    :target: http://travis-ci.org/ionelmc/python-manhole

.. image:: https://coveralls.io/repos/ionelmc/python-manhole/badge.png?branch=master
    :alt: Coverage Status
    :target: https://coveralls.io/r/ionelmc/python-manhole

Coverage is wrong, must be a bug in coveralls, it should be at least 80%-90% depending whether you count branches or not.

Similar projects
================

* Twisted's `old manhole <http://twistedmatrix.com/documents/current/api/twisted.manhole.html>`__ and the `newer implementation <http://twistedmatrix.com/documents/current/api/twisted.conch.manhole.html>`__ (colors, serverside history).
* `wsgi-shell <https://github.com/GrahamDumpleton/wsgi-shell>`_ - spawns a thread.
* `pyrasite <https://github.com/lmacken/pyrasite>`_ - uses gdb to inject code.
* `pydbattach <https://github.com/albertz/pydbattach>`_ - uses gdb to inject code.
