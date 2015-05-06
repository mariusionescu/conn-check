conn-check
==========

conn-check allows for checking connectivity with external services.

You can write a config file that defines services that you need to
have access to, and conn-check will check connectivity with each.

It supports various types of services, all of which allow for
basic network checks, but some allow for confirming credentials
work also.

Configuration
-------------

The configuration is done via a yaml file. The file defines a list
of checks to do::

    - type: tcp
      host: localhost
      port: 80
    - type: tls
      host: localhost
      port: 443
      disable_tls_verification: false

Each check defines a type, and then options as appropriate for that type.

Check Types
-----------

tcp
```

A simple tcp connectivity check.

host
    The host.

port
    The port.

timeout
    Optional connection timeout in seconds. Default: 10 (or value from ``--connect-timeout``).


tls
```

A check that uses TLS (`ssl` is a deprecated alias for this type).

host
    The host.

port
    The port.

disable_tls_verification
    Optional flag to disable verification of TLS certs and handshake. Default:
    false.

timeout
    Optional connection timeout in seconds. Default: 10 (or value from ``--connect-timeout``).


udp
```

Check that sending a specific UDP packet gets a specific response.

host
    The host.

port
    The port.

send
    The string to send.

expect
    The string to expect in the response.

timeout
    Optional connection timeout in seconds. Default: 10 (or value from ``--connect-timeout``).


http
````

Check that a HTTP/HTTPS request succeeds (`https` also works).

url
    The URL to fetch.

method
    Optional HTTP method to use. Default: "GET".

expected_code
    Optional status code that defines success. Default: 200.

proxy_url
    Optional HTTP/HTTPS proxy URL to connect via, including protocol,
    if set proxy_{host,port} are ignored.

proxy_host
    Optional HTTP/HTTPS proxy to connect via.

proxy_port
    Optional port to use with ``proxy_host``. Default: 8000.

headers:
    Optional headers to send, as a dict of key-values. Multiple values can be
    given as a list/tuple of lists/tuples, e.g.:
    ``[('foo', 'bar'), ('foo', 'baz')]``

body:
    Optional raw request body string to send.

disable_tls_verification:
    Optional flag to disable verification of TLS certs and handshake. Default:
    false.

timeout
    Optional connection timeout in seconds. Default: 10 (or value from ``--connect-timeout``).

allow_redirects
    Optional flag to Follow 30x redirects. Default: false.

params
    Optional dict of params to URL encode and pass in the querystring.

cookies
    Optional dict of cookies to pass in the request headers.

auth
    Optional `basic HTTP auth <https://en.wikipedia.org/wiki/Basic_access_authentication>`_
    credentials, as a tuple/list: ``(username, password)``.

digest_auth
    Optional `digest HTTP auth <https://en.wikipedia.org/wiki/Digest_access_authentication>`_
    credentials, as a tuple/list: ``(username, password)``.


amqp
````

Check that an AMQP server can be authenticated against.

host
    The host.

port
    The port.

username
    The username to authenticate with.

password
    The password to authenticate with.

use_tls
    Optional flag whether to connect with TLS. Default: true.

vhost
    Optional vhost name to connect to. Default '/'.

timeout
    Optional connection timeout in seconds. Default: 10 (or value from ``--connect-timeout``).


postgres
````````

Check that a PostgreSQL db can be authenticated against (`postgresql` also works).

host
    The host.

port
    The port.

username
    The username to authenticate with.

password
    The password to authenticate with.

database
    The database to connect to.

timeout
    Optional connection timeout in seconds. Default: 10 (or value from ``--connect-timeout``).


redis
`````

Check that a redis server is present, optionally checking authentication.

host
    The host.

port
    The port.

password
    Optional password to authenticatie with.

timeout
    Optional connection timeout in seconds. Default: 10 (or value from ``--connect-timeout``).


memcache
````````

Check that a memcached server is present (`memcached` also works).

host
    The host.

port
    The port.

timeout
    Optional connection timeout in seconds. Default: 10 (or value from ``--connect-timeout``).


mongodb
```````

Check that a MongoDB server is present (`mongo` also works).

host
    The host.

port
    Optional port. Default: 27017.

username
    Optional username to authenticate with.

password
    Optional password to authenticate with.

database
    Optional database name to connect to, if not set the ``test`` database will be used,
    if this database does not exist (or is not available to the user) you will need to
    provide a database name.

timeout
    Optional connection timeout in seconds. Default: 10 (or value from ``--connect-timeout``).


Tags
----

Every check type also supports a ``tags`` field, which is a list of tags that
can be used with the ``--include-tags`` and ``--exclude-tags`` arguments to conn-check.

Example YAML::

    - type: http
      url: http://google.com/
      tags:
        - external

To run just "external" checks::

    conn-check --include-tags=external ...

To run all the checks *except* external::

    conn-check --exclude-tags=external

Buffered/Ordered output
-----------------------

conn-check normally executes with output to ``STDOUT`` buffered so that the output can be ordered,
with failed checks being printed first, grouping by destination etc.

If you'd rather see results as they available you can use the ``-U``/``--unbuffered-output`` option
to disable buffering.

Building wheels
---------------

To allow for easier/more portable distribution of this tool you can build
conn-check and all it's dependencies as `Python wheels <http://legacy.python.org/dev/peps/pep-0427/>`_::

    make clean-wheels
    make build-wheels
    make build-wheels-extra EXTRA=amqp
    make build-wheels-extra EXTRA=redis

The `build-wheels` make target will build conn-check and it's base
dependencies, but to include the optional extra dependencies for other
checks such as amqp, redis or postgres you need to use the
`build-wheels-extra` target with the `EXTRA` env value.

By default all the wheels will be placed in `./wheels`.


Automatically generating conn-check YAML configurations
-------------------------------------------------------

The `conn-check-configs <https://pypi.python.org/pypi/conn-check-configs>`_ package contains utilities/libraries
for generating checks from existing application configurations and environments, e.g. from Django settings modules
and Juju environments.
