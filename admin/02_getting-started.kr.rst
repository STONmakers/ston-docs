.. _getting-started:

STON 시작하기
******************

STON은 모든 것이 간단합니다. 빠르게 설치해서 바로 서비스에 투입할 수 있습니다. 이 장에서는 설치부터 실행까지의 내용을 다룹니다.

.. toctree::
   :maxdepth: 2

서버 구성
====================================

STON은 시장에서 가장 저렴하게 구입할 수 있는 장비에서도 무리없이 높은 퍼포먼스를 보장하도록 개발되었습니다. 고객이 합리적인 장비 선택을 할 수 있도록 돕는 것 또한 STON의 목표입니다. 서버는 서비스의 특성과 규모에 따라 다르게 구성될 수 있습니다. 일반적인 선택의 기준은 CPU, 메모리, 디스크입니다.

-  **CPU**

   STON은 Many-Core에 대해 확장성(Scalability)를 가집니다. 코어가 많으면 많을수록 
   초당 처리량이 증가합니다. 높은 처리량이 반드시 높은 트래픽을 의미하는 것은 아닙니다.

   .. figure:: img/10g_cpu.jpg
      :align: center

      클라이언트가 많을수록 많은 CPU는 힘이 됩니다.
    
   위 그림처럼 평균파일 크기가 4KB인 서비스는 약 26만번을 전송해야 1GB파일을 
   한번 전송한 것과 같은 대역폭을 사용합니다. CPU선택의 가장 큰 기준은 얼마나 
   많은 동시접속을 처리하는가에 있습니다. 일반적으로 Quad 코어 이상이 적합합니다. 
   

-  **메모리**

   최소 4GB이상을 권장합니다. 자주 접근되는 콘텐츠를 메모리에 상주시켜야 
   성능이 향상되는데, 물리적인 메모리가 작다면 디스크 부하가 가중됩니다.
   파일이 많지 않더라도 콘텐츠 크기가 커서 디스크 I/O가 높다면 메모리를 
   증설하는 것이 좋은 해결책입니다.


-  **디스크**

   OS를 포함하여 최소 3개 이상을 권장합니다. 디스크 역시 많으면 많을수록 많은 
   콘텐츠를 캐싱할 수 있을 뿐만 아니라 부하를  분산할 수 있으므로 좋습니다.
   
   .. figure:: img/02_disk.png
      :align: center
      
   OS와 STON은 별도의 디스크로 구성하시기 바랍니다.
   
   일반적으로 OS가 설치된 디스크에 STON을 설치합니다. 로그 역시 같은 디스크에
   구성하는 것이 일반적입니다. (로그의 경우 모든 서비스 상황을 실시간으로 기록하기 
   때문에 항상 Write작업이 있습니다.) STON은 디스크를 RAID 0처럼 사용합니다. 
   퍼포먼스와 RAID의 상관여부는 고객 서비스 특성에 따라 달라지므로 단언할 수 없습니다. 
   파일 변경이 빈번하지 않고 컨텐츠의 크기가 메모리 캐싱크기보다 훨씬 큰 경우 
   RAID를 통한 Read속도 향상이 도움이 될 수 있습니다.



OS 구성
================================

표준 64bit Linux 배포판(Cent 6.2이상, Ubuntu 10.04이상)에서 동작합니다.
가장 기본적인 형태로 설치하시면 됩니다. STON은 다른 패키지에 종속성을 
가지지 않으므로 추가로 설치하실 것은 없습니다.


설치
====================================

1. 최신버전의 STON을 다운로드 받는다.

   [root@localhost ~]# wget  http://webhard.winesoft.co.kr/ston/ston.1.3.11.rhel.2.6.32.x64.tar.gz
   --2014-06-17 13:29:14--  http://webhard.winesoft.co.kr/ston/ston.1.3.11.rhel.2.6.32.x64.tar.gz
   Resolving webhard.winesoft.co.kr... 192.168.0.14
   Connecting to webhard.winesoft.co.kr|192.168.0.14|:80... connected.
   HTTP request sent, awaiting response... 200 OK
   Length: 71340645 (68M) [application/x-gzip]
   Saving to: “ston.1.3.11.rhel.2.6.32.x64.tar.gz”

   100%[===============================================>] 71,340,645  42.9M/s   in 1.6s

   2014-06-17 13:29:15 (42.9 MB/s) - “ston.1.3.11.rhel.2.6.32.x64.tar.gz” saved [71340645/71340645]


2. 압축을 해지합니다.

3. 

   Some HTTP objects contain ``Expires`` headers or ``max-age`` headers
   that explicitly define how long the object can be cached. Traffic
   Server compares the current time with the expiration time to
   determine if the object is still fresh.

-  **Checking the** ``Last-Modified`` **/** ``Date`` **header**

   If an HTTP object has no ``Expires`` header or ``max-age`` header,
   then Traffic Server can calculate a freshness limit using the
   following formula::

      freshness_limit = ( date - last_modified ) * 0.10

   where *date* is the date in the object's server response header
   and *last_modified* is the date in the ``Last-Modified`` header.
   If there is no ``Last-Modified`` header, then Traffic Server uses the
   date the object was written to cache. The value ``0.10`` (10 percent)
   can be increased or reduced to better suit your needs (refer to
   `Modifying Aging Factor for Freshness Computations`_).

   The computed freshness limit is bound by a minimum and maximum value
   - refer to `Setting Absolute Freshness Limits`_ for more information.

-  **Checking the absolute freshness limit**

   For HTTP objects that do not have ``Expires`` headers or do not have
   both ``Last-Modified`` and ``Date`` headers, Traffic Server uses a
   maximum and minimum freshness limit (refer to `Setting Absolute Freshness Limits`_).

-  **Checking revalidate rules in the** :file:`cache.config` **file**

   Revalidate rules apply freshness limits to specific HTTP objects. You
   can set freshness limits for objects originating from particular
   domains or IP addresses, objects with URLs that contain specified
   regular expressions, objects requested by particular clients, and so
   on (refer to :file:`cache.config`).

Modifying Aging Factor for Freshness Computations
-------------------------------------------------

If an object does not contain any expiration information, then Traffic
Server can estimate its freshness from the ``Last-Modified`` and
``Date`` headers. By default, Traffic Server stores an object for 10% of
the time that elapsed since it last changed. You can increase or reduce
the percentage according to your needs.

To modify the aging factor for freshness computations

1. Change the value for :ts:cv:`proxy.config.http.cache.heuristic_lm_factor`.

2. Run the :option:`traffic_line -x` command to apply the configuration
   changes.

Setting absolute Freshness Limits
---------------------------------

Some objects do not have ``Expires`` headers or do not have both
``Last-Modified`` and ``Date`` headers. To control how long these
objects are considered fresh in the cache, specify an **absolute
freshness limit**.

To specify an absolute freshness limit

1. Edit the variables

   -  :ts:cv:`proxy.config.http.cache.heuristic_min_lifetime`
   -  :ts:cv:`proxy.config.http.cache.heuristic_max_lifetime`

2. Run the :option:`traffic_line -x` command to apply the configuration
   changes.

Specifying Header Requirements
------------------------------

To further ensure freshness of the objects in the cache, configure
Traffic Server to cache only objects with specific headers. By default,
Traffic Server caches all objects (including objects with no headers);
you should change the default setting only for specialized proxy
situations. If you configure Traffic Server to cache only HTTP objects
with ``Expires`` or ``max-age`` headers, then the cache hit rate will be
noticeably reduced (since very few objects will have explicit expiration
information).

To configure Traffic Server to cache objects with specific headers

1. Change the value for :ts:cv:`proxy.config.http.cache.required_headers`.

2. Run the :option:`traffic_line -x` command to apply the configuration
   changes.

.. _cache-control-headers:

Cache-Control Headers
---------------------

Even though an object might be fresh in the cache, clients or servers
often impose their own constraints that preclude retrieval of the object
from the cache. For example, a client might request that a object *not*
be retrieved from a cache, or if it does, then it cannot have been
cached for more than 10 minutes. Traffic Server bases the servability of
a cached object on ``Cache-Control`` headers that appear in both client
requests and server responses. The following ``Cache-Control`` headers
affect whether objects are served from cache:

-  The ``no-cache`` header, sent by clients, tells Traffic Server that
   it should not serve any objects directly from the cache;
   therefore, Traffic Server will always obtain the object from the
   origin server. You can configure Traffic Server to ignore client
   ``no-cache`` headers - refer to `Configuring Traffic Server to Ignore Client no-cache Headers`_
   for more information.

-  The ``max-age`` header, sent by servers, is compared to the object
   age. If the age is less than ``max-age``, then the object is fresh
   and can be served.

-  The ``min-fresh`` header, sent by clients, is an **acceptable
   freshness tolerance**. This means that the client wants the object to
   be at least this fresh. Unless a cached object remains fresh at least
   this long in the future, it is revalidated.

-  The ``max-stale`` header, sent by clients, permits Traffic Server to
   serve stale objects provided they are not too old. Some browsers
   might be willing to take slightly stale objects in exchange for
   improved performance, especially during periods of poor Internet
   availability.

Traffic Server applies ``Cache-Control`` servability criteria
***after*** HTTP freshness criteria. For example, an object might be
considered fresh but will not be served if its age is greater than its
``max-age``.

Revalidating HTTP Objects
-------------------------

When a client requests an HTTP object that is stale in the cache,
Traffic Server revalidates the object. A **revalidation** is a query to
the origin server to check if the object is unchanged. The result of a
revalidation is one of the following:

-  If the object is still fresh, then Traffic Server resets its
   freshness limit and serves the object.

-  If a new copy of the object is available, then Traffic Server caches
   the new object (thereby replacing the stale copy) and simultaneously
   serves the object to the client.

-  If the object no longer exists on the origin server, then Traffic
   Server does not serve the cached copy.

-  If the origin server does not respond to the revalidation query, then
   Traffic Server serves the stale object along with a
   ``111 Revalidation Failed`` warning.

By default, Traffic Server revalidates a requested HTTP object in the
cache if it considers the object to be stale. Traffic Server evaluates
object freshness as described in `HTTP Object Freshness`_.
You can reconfigure how Traffic Server evaluates freshness by selecting
one of the following options:

-  Traffic Server considers all HTTP objects in the cache to be stale:
   always revalidate HTTP objects in the cache with the origin server.
-  Traffic Server considers all HTTP objects in the cache to be fresh:
   never revalidate HTTP objects in the cache with the origin server.
-  Traffic Server considers all HTTP objects without ``Expires`` or
   ``Cache-control`` headers to be stale: revalidate all HTTP objects
   without ``Expires`` or ``Cache-Control`` headers.

To configure how Traffic Server revalidates objects in the cache, you
can set specific revalidation rules in :file:`cache.config`.

To configure revalidation options

1. Edit the following variable in :file:`records.config`

   -  :ts:cv:`proxy.config.http.cache.when_to_revalidate`

2. Run the :option:`traffic_line -x` command to apply the configuration
   changes.

Scheduling Updates to Local Cache Content
=========================================

To further increase performance and to ensure that HTTP objects are
fresh in the cache, you can use the **Scheduled Update** option. This
configures Traffic Server to load specific objects into the cache at
scheduled times. You might find this especially beneficial in a reverse
proxy setup, where you can *preload* content you anticipate will be in
demand.

To use the Scheduled Update option, you must perform the following
tasks.

-  Specify the list of URLs that contain the objects you want to
   schedule for update,
-  the time the update should take place,
-  and the recursion depth for the URL.
-  Enable the scheduled update option and configure optional retry
   settings.

Traffic Server uses the information you specify to determine URLs for
which it is responsible. For each URL, Traffic Server derives all
recursive URLs (if applicable) and then generates a unique URL list.
Using this list, Traffic Server initiates an HTTP ``GET`` for each
unaccessed URL. It ensures that it remains within the user-defined
limits for HTTP concurrency at any given time. The system logs the
completion of all HTTP ``GET`` operations so you can monitor the
performance of this feature.

Traffic Server also provides a **Force Immediate Update** option that
enables you to update URLs immediately without waiting for the specified
update time to occur. You can use this option to test your scheduled
update configuration (refer to `Forcing an Immediate Update`_).

Configuring the Scheduled Update Option
---------------------------------------

To configure the scheduled update option

1. Edit :file:`update.config` to
   enter a line in the file for each URL you want to update.
2. Edit the following variables

   -  :ts:cv:`proxy.config.update.enabled`
   -  :ts:cv:`proxy.config.update.retry_count`
   -  :ts:cv:`proxy.config.update.retry_interval`
   -  :ts:cv:`proxy.config.update.concurrent_updates`

3. Run the :option:`traffic_line -x` command to apply the configuration
   changes.

Forcing an Immediate Update
---------------------------

Traffic Server provides a **Force Immediate Update** option that enables
you to immediately verify the URLs listed in :file:`update.config`.
The Force Immediate Update option disregards the offset hour and
interval set in :file:`update.config` and immediately updates the
URLs listed.

To configure the Force Immediate Update option

1. Edit the following variables

   -  :ts:cv:`proxy.config.update.force`
   -  Make sure :ts:cv:`proxy.config.update.enabled` is set to 1.

2. Run the command :option:`traffic_line -x` to apply the configuration
   changes.

.. important::

   When you enable the Force Immediate Update option, Traffic Server continually updates the URLs specified in
   :file:`update.config` until you disable the option. To disable the Force Immediate Update option, set
   :ts:cv:`proxy.config.update.force` to ``0`` (zero).

Pushing Content into the Cache
==============================

Traffic Server supports the HTTP ``PUSH`` method of content delivery.
Using HTTP ``PUSH``, you can deliver content directly into the cache
without client requests.

Configuring Traffic Server for PUSH Requests
--------------------------------------------

Before you can deliver content into your cache using HTTP ``PUSH``, you
must configure Traffic Server to accept ``PUSH`` requests.

To configure Traffic Server to accept ``PUSH`` requests

1. Edit :file:`ip_allow.config` to allow ``PUSH``.

2. Edit the following variable in :file:`records.config`, enable
   the push_method.

   -  :ts:cv:`proxy.config.http.push_method_enabled`

3. Run the command :option:`traffic_line -x` to apply the configuration
   changes.

Understanding HTTP PUSH
-----------------------

``PUSH`` uses the HTTP 1.1 message format. The body of a ``PUSH``
request contains the response header and response body that you want to
place in the cache. The following is an example of a ``PUSH`` request::

   PUSH http://www.company.com HTTP/1.0
   Content-length: 84

   HTTP/1.0 200 OK
   Content-type: text/html
   Content-length: 17

   <HTML>
   a
   </HTML>

.. important::

   Your header must include ``Content-length`` - ``Content-length`` must include both ``header`` and ``body byte
   count``.

Tools that will help manage pushing
-----------------------------------

There is a perl script for pushing, :program:`tspush`,
which can help you understanding how to write scripts for pushing
content yourself.

Pinning Content in the Cache
============================

The **Cache Pinning Option** configures Traffic Server to keep certain
HTTP objects in the cache for a specified time. You can use this option
to ensure that the most popular objects are in cache when needed and to
prevent Traffic Server from deleting important objects. Traffic Server
observes ``Cache-Control`` headers and pins an object in the cache only
if it is indeed cacheable.

To set cache pinning rules

1. Make sure the following variable in :file:`records.config` is set

   -  :ts:cv:`proxy.config.cache.permit.pinning`

2. Add a rule in :file:`cache.config` for each
   URL you want Traffic Server to pin in the cache. For example::

      url_regex=^https?://(www.)?apache.org/dev/ pin-in-cache=12h

3. Run the command :option:`traffic_line -x` to apply the configuration
   changes.

To Cache or Not to Cache?
=========================

When Traffic Server receives a request for a web object that is not in
the cache, it retrieves the object from the origin server and serves it
to the client. At the same time, Traffic Server checks if the object is
cacheable before storing it in its cache to serve future requests.

Caching HTTP Objects
====================

Traffic Server responds to caching directives from clients and origin
servers, as well as directives you specify through configuration options
and files.

Client Directives
-----------------

By default, Traffic Server does *not* cache objects with the following
**request headers**:

-  ``Authorization``: header

-  ``Cache-Control: no-store`` header

-  ``Cache-Control: no-cache`` header

   To configure Traffic Server to ignore the ``Cache-Control: no-cache``
   header, refer to `Configuring Traffic Server to Ignore Client no-cache Headers`_

-  ``Cookie``: header (for text objects)

   By default, Traffic Server caches objects served in response to
   requests that contain cookies (unless the object is text). You can
   configure Traffic Server to not cache cookied content of any type,
   cache all cookied content, or cache cookied content that is of image
   type only. For more information, refer to `Caching Cookied Objects`_.

Configuring Traffic Server to Ignore Client no-cache Headers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

By default, Traffic Server strictly observes client
``Cache-Control: no-cache`` directives. If a requested object contains a
``no-cache`` header, then Traffic Server forwards the request to the
origin server even if it has a fresh copy in cache. You can configure
Traffic Server to ignore client ``no-cache`` directives such that it
ignores ``no-cache`` headers from client requests and serves the object
from its cache.

To configure Traffic Server to ignore client ``no-cache`` headers

1. Edit the following variable in :file:`records.config`

   -  :ts:cv:`proxy.config.http.cache.ignore_client_no_cache`

2. Run the command :option:`traffic_line -x` to apply the configuration
   changes.

Origin Server Directives
------------------------

By default, Traffic Server does *not* cache objects with the following
**response headers**:

-  ``Cache-Control: no-store`` header
-  ``Cache-Control: private`` header
-  ``WWW-Authenticate``: header

   To configure Traffic Server to ignore ``WWW-Authenticate`` headers,
   refer to `Configuring Traffic Server to Ignore WWW-Authenticate Headers`_.

-  ``Set-Cookie``: header
-  ``Cache-Control: no-cache`` headers

   To configure Traffic Server to ignore ``no-cache`` headers, refer to
   `Configuring Traffic Server to Ignore Server no-cache Headers`_.

-  ``Expires``: header with value of 0 (zero) or a past date

Configuring Traffic Server to Ignore Server no-cache Headers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

By default, Traffic Server strictly observes ``Cache-Control: no-cache``
directives. A response from an origin server with a ``no-cache`` header
is not stored in the cache and any previous copy of the object in the
cache is removed. If you configure Traffic Server to ignore ``no-cache``
headers, then Traffic Server also ignores ``no-store`` headers. The
default behavior of observing ``no-cache`` directives is appropriate
in most cases.

To configure Traffic Server to ignore server ``no-cache`` headers

#. Edit the variable :ts:cv:`proxy.config.http.cache.ignore_server_no_cache`

#. Run the command :option:`traffic_line -x` to apply the configuration
   changes.

Configuring Traffic Server to Ignore WWW-Authenticate Headers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

By default, Traffic Server does not cache objects that contain
``WWW-Authenticate`` response headers. The ``WWW-Authenticate`` header
contains authentication parameters the client uses when preparing the
authentication challenge response to an origin server.

When you configure Traffic Server to ignore origin server
``WWW-Authenticate`` headers, all objects with ``WWW-Authenticate``
headers are stored in the cache for future requests. However, the
default behavior of not caching objects with ``WWW-Authenticate``
headers is appropriate in most cases. Only configure Traffic Server to
ignore server ``WWW-Authenticate`` headers if you are knowledgeable
about HTTP 1.1.

To configure Traffic Server to ignore server ``WWW-Authenticate``
headers

#. Edit the variable :ts:cv:`proxy.config.http.cache.ignore_authentication`

#. Run the command :option:`traffic_line -x` to apply the configuration
   changes.

Configuration Directives
------------------------

In addition to client and origin server directives, Traffic Server
responds to directives you specify through configuration options and
files.

You can configure Traffic Server to do the following:

-  *Not* cache any HTTP objects (refer to `Disabling HTTP Object Caching`_).
-  Cache **dynamic content** - that is, objects with URLs that end in
   ``.asp`` or contain a question mark (``?``), semicolon
   (**``;``**), or **``cgi``**. For more information, refer to `Caching Dynamic Content`_.
-  Cache objects served in response to the ``Cookie:`` header (refer to
   `Caching Cookied Objects`_.
-  Observe ``never-cache`` rules in the :file:`cache.config` file.

Disabling HTTP Object Caching
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

By default, Traffic Server caches all HTTP objects except those for
which you have set ``never-cache`` as :ref:`action rules <cache-config-format-action>`
in the :file:`cache.config` file. You can disable HTTP object
caching so that all HTTP objects are served directly from the origin
server and never cached, as detailed below.

To disable HTTP object caching manually

1. Set the variable :ts:cv:`proxy.config.http.enabled` to ``0``.

2. Run the command :option:`traffic_line -x` to apply the configuration
   changes.

Caching Dynamic Content
~~~~~~~~~~~~~~~~~~~~~~~

A URL is considered **dynamic** if it ends in **``.asp``** or contains a
question mark (``?``), a semicolon (``;``), or ``cgi``. By
default, Traffic Server caches dynamic content. You can configure the
system to ignore dyanamic looking content, although this is recommended
only if the content is *truely* dyanamic, but fails to advertise so with
appropriate ``Cache-Control`` headers.

To configure Traffic Server's cache behaviour in regard to dynamic
content

1. Edit the following variable in :file:`records.config`

   -  :ts:cv:`proxy.config.http.cache.cache_urls_that_look_dynamic`

2. Run the command :option:`traffic_line -x` to apply the configuration
   changes.

Caching Cookied Objects
~~~~~~~~~~~~~~~~~~~~~~~

.. XXX This should be extended to xml as well!

By default, Traffic Server caches objects served in response to requests
that contain cookies. This is true for all types of objects except for
text. Traffic Server does not cache cookied text content because object
headers are stored along with the object, and personalized cookie header
values could be saved with the object. With non-text objects, it is
unlikely that personalized headers are delivered or used.

You can reconfigure Traffic Server to:

-  *Not* cache cookied content of any type.
-  Cache cookied content that is of image type only.
-  Cache all cookied content regardless of type.

To configure how Traffic Server caches cookied content

1. Edit the variable :ts:cv:`proxy.config.http.cache.cache_responses_to_cookies`

2. Run the command :option:`traffic_line -x` to apply the configuration
   changes.

Forcing Object Caching
======================

You can force Traffic Server to cache specific URLs (including dynamic
URLs) for a specified duration, regardless of ``Cache-Control`` response
headers.

To force document caching

1. Add a rule for each URL you want Traffic Server to pin to the cache
   :file:`cache.config`::

       url_regex=^https?://(www.)?apache.org/dev/ ttl-in-cache=6h

2. Run the command :option:`traffic_line -x` to apply the configuration
   changes.

Caching HTTP Alternates
=======================

Some origin servers answer requests to the same URL with a variety of
objects. The content of these objects can vary widely, according to
whether a server delivers content for different languages, targets
different browsers with different presentation styles, or provides
different document formats (HTML, XML). Different versions of the same
object are termed **alternates** and are cached by Traffic Server based
on ``Vary`` response headers. You can specify additional request and
response headers for specific ``Content-Type``\s that Traffic Server
will identify as alternates for caching. You can also limit the number
of alternate versions of an object allowed in the cache.

Configuring How Traffic Server Caches Alternates
------------------------------------------------

To configure how Traffic Server caches alternates, follow the steps
below

1. Edit the following variables

   -  :ts:cv:`proxy.config.http.cache.enable_default_vary_headers`
   -  :ts:cv:`proxy.config.http.cache.vary_default_text`
   -  :ts:cv:`proxy.config.http.cache.vary_default_images`
   -  :ts:cv:`proxy.config.http.cache.vary_default_other`

2. Run the command :option:`traffic_line -x` to apply the configuration
   changes.

.. note::

   If you specify ``Cookie`` as the header field on which to vary
   in the above variables, make sure that the variable
   :ts:cv:`proxy.config.http.cache.cache_responses_to_cookies`
   is set appropriately.

Limiting the Number of Alternates for an Object
-----------------------------------------------

You can limit the number of alternates Traffic Server can cache per
object (the default is 3).

.. important::

   Large numbers of alternates can affect Traffic Server
   cache performance because all alternates have the same URL. Although
   Traffic Server can look up the URL in the index very quickly, it must
   scan sequentially through available alternates in the object store.

   To limit the number of alternates

   #. Edit the variable :ts:cv:`proxy.config.cache.limits.http.max_alts`
   #. Run the command :option:`traffic_line -x` to apply the configuration changes.


.. _using-congestion-control:

Using Congestion Control
========================

The **Congestion Control** option enables you to configure Traffic
Server to stop forwarding HTTP requests to origin servers when they
become congested. Traffic Server then sends the client a message to
retry the congested origin server later.

To use the **Congestion Control** option, you must perform the following
tasks:

#. Set the variable :ts:cv:`proxy.config.http.congestion_control.enabled` to ``1``

   -  Create rules in the :file:`congestion.config` file to specify:
   -  which origin servers Traffic Server tracks for congestion
   -  the timeouts Traffic Server uses, depending on whether a server is
      congested
   -  the page Traffic Server sends to the client when a server becomes
      congested
   -  if Traffic Server tracks the origin servers per IP address or per
      hostname

#. Run the command :option:`traffic_line -x` to apply the configuration
   changes.

.. _transaction-buffering-control:

Using Transaction Buffering Control
===================================

By default I/O operations are run at full speed, as fast as either Traffic Server, the network, or the cache can go.
This can be problematic for large objects if the client side connection is significantly slower. In such cases the
content will be buffered in ram while waiting to be sent to the client. This could potentially also happen for ``POST``
requests if the client connection is fast and the origin server connection slow. If very large objects are being used
this can cause the memory usage of Traffic Server to become `very large
<https://issues.apache.org/jira/browse/TS-1496>`_.

This problem can be ameloriated by controlling the amount of buffer space used by a transaction. A high water and low
water mark are set in terms of bytes used by the transaction. If the buffer space in use exceeds the high water mark,
the connection is throttled to prevent additional external data from arriving. Internal operations continue to proceed
at full speed until the buffer space in use drops below the low water mark and external data I/O is re-enabled.

Although this is intended primarily to limit the memory usage of Traffic Server it can also serve as a crude rate
limiter by setting a buffer limit and then throttling the client side connection either externally or via a transform.
This will cause the connection to the origin server to be limited to roughly the client side connection speed.

Traffic Server does network I/O in large chunks (32K or so) and therefore the granularity of transaction buffering
control is limited to a similar precision.

The buffer size calculations include all elements in the transaction, including any buffers associated with :ref:`transform plugins <transform-plugin>`.

Transaction buffering control can be enabled globally by using configuration variables or by :c:func:`TSHttpTxnConfigIntSet` in a plugin.

================= ================================================== ================================================
Value             Variable                                           :c:func:`TSHttpTxnConfigIntSet` key
================= ================================================== ================================================
Enable buffering  :ts:cv:`proxy.config.http.flow_control.enabled`    :c:data:`TS_CONFIG_HTTP_FLOW_CONTROL_ENABLED`
Set high water    :ts:cv:`proxy.config.http.flow_control.high_water` :c:data:`TS_CONFIG_HTTP_FLOW_CONTROL_HIGH_WATER`
Set low water     :ts:cv:`proxy.config.http.flow_control.low_water`  :c:data:`TS_CONFIG_HTTP_FLOW_CONTROL_LOW_WATER`
================= ================================================== ================================================

Be careful to always have the low water mark equal or less than the high water mark. If you set only one, the other will
be set to the same value.

If using :c:func:`TSHttpTxnConfigIntSet`, it must be called no later than :c:data:`TS_HTTP_READ_RESPONSE_HDR_HOOK`.

.. _reducing-origin-server-requests-avoiding-the-thundering-herd:

Reducing Origin Server Requests (Avoiding the Thundering Herd)
==============================================================

When an object can not be served from cache, the request will be proxied to the origin server. For a popular object,
this can result in many near simultaneous requests to the origin server, potentially overwhelming it or associated
resources. There are several features in Traffic Server that can be used to avoid this scenario.

Read While Writer
-----------------
When Traffic Server goes to fetch something from origin, and upon receiving the response, any number of clients can be allowed to start serving the partially filled cache object once background_fill_completed_threshold % of the object has been received. The difference is that Squid allows this as soon as it goes to origin, whereas ATS can not do it until we get the complete response header. The reason for this is that we make no distinction between cache refresh, and cold cache, so we have no way to know if a response is going to be cacheable, and therefore allow read-while-writer functionality.

The configurations necessary to enable this in ATS are:

|   CONFIG :ts:cv:`proxy.config.cache.enable_read_while_writer` ``INT 1``
|   CONFIG :ts:cv:`proxy.config.http.background_fill_active_timeout` ``INT 0``
|   CONFIG :ts:cv:`proxy.config.http.background_fill_completed_threshold` ``FLOAT 0.000000``
|   CONFIG :ts:cv:`proxy.config.cache.max_doc_size` ``INT 0`` 

All four configurations are required, for the following reasons:

-  enable_read_while_writer turns the feature on. It's off (0) by default
-  The background fill feature should be allowed to kick in for every possible request. This is necessary, in case the writer ("first client session") goes away, someone needs to take over the session. Hence, you should set the background fill timeouts and threshold to zero; this assures they never times out and always is allowed to kick in. 
-  The proxy.config.cache.max_doc_size should be unlimited (set to 0), since the object size may be unknown, and going over this limit would cause a disconnect on the objects being served.

Once all this enabled, you have something that is very close, but not quite the same, as Squid's Collapsed Forwarding.



.. _fuzzy-revalidation:

Fuzzy Revalidation
------------------
Traffic Server can be set to attempt to revalidate an object before it becomes stale in cache. :file:`records.config` contains the settings:

|   CONFIG :ts:cv:`proxy.config.http.cache.fuzz.time` ``INT 240``
|   CONFIG :ts:cv:`proxy.config.http.cache.fuzz.min_time` ``INT 0``
|   CONFIG :ts:cv:`proxy.config.http.cache.fuzz.probability` ``FLOAT 0.005``

For every request for an object that occurs "fuzz.time" before (in the example above, 240 seconds) the object is set to become stale, there is a small
chance (fuzz.probability == 0.5%) that the request will trigger a revalidation request to the origin. For objects getting a few requests per second, this would likely not trigger, but then this feature is not necessary anyways since odds are only 1 or a small number of connections would hit origin upon objects going stale. The defaults are a good compromise, for objects getting roughly 4 requests / second or more, it's virtually guaranteed to trigger a revalidate event within the 240s. These configs are also overridable per remap rule or via a plugin, so can be adjusted per request if necessary.  

Note that if the revalidation occurs, the requested object is no longer available to be served from cache.  Subsequent
requests for that object will be proxied to the origin. 

Finally, the fuzz.min_time is there to be able to handle requests with a TTL less than fuzz.time ? it allows for different times to evaluate the probability of revalidation for small TTLs and big TTLs. Objects with small TTLs will start "rolling the revalidation dice" near the fuzz.min_time, while objects with large TTLs would start at fuzz.time. A logarithmic like function between determines the revalidation evaluation start time (which will be between fuzz.min_time and fuzz.time). As the object gets closer to expiring, the window start becomes more likely. By default this setting is not enabled, but should be enabled anytime you have objects with small TTLs. Note that this option predates overridable configurations, so you can achieve something similar with a plugin or remap.config conf_remap.so configs.

These configurations are similar to Squid's refresh_stale_hit configuration option.


Open Read Retry Timeout
-----------------------

The open read retry configurations attempt to reduce the number of concurrent requests to the origin for a given object. While an object is being fetched from the origin server, subsequent requests would wait open_read_retry_time milliseconds before checking if the object can be served from cache. If the object is still being fetched, the subsequent requests will retry max_open_read_retries times. Thus, subsequent requests may wait a total of (max_open_read_retries x open_read_retry_time) milliseconds before establishing an origin connection of its own. For instance, if they are set to 5 and 10 respectively, connections will wait up to 50ms for a response to come back from origin from a previous request, until this request is allowed through.

These settings are inappropriate when objects are uncacheable. In those cases, requests for an object effectively become serialized. The subsequent requests would await at least open_read_retry_time milliseconds before being proxies to the origin.

Similarly, this setting should be used in conjunction with Read While Writer for big (those that take longer than (max_open_read_retries x open_read_retry_time) milliseconds to transfer) cacheable objects. Without the read-while-writer settings enabled, while the initial fetch is ongoing, not only would subsequent requests be delayed by the maximum time, but also, those requests would result in another request to the origin server.

Since ATS now supports setting these settings per-request or remap rule, you can configure this to be suitable for your setup much more easily.

The configurations are (with defaults):

|   CONFIG :ts:cv:`proxy.config.http.cache.max_open_read_retries` ``INT -1``
|   CONFIG :ts:cv:`proxy.config.http.cache.open_read_retry_time` ``INT 10``

The default means that the feature is disabled, and every connection is allowed to go to origin instantly. When enabled, you will try max_open_read_retries times, each with a open_read_retry_time timeout.
