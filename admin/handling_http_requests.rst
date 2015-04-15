﻿.. _handling_http_requests:

6장. HTTP 요청처리
******************

이 장에서는 HTTP 클라이언트 세션과 요청을 처리하는 방식에 대해 설명한다.
서비스의 핵심기능으로 보기엔 어려운 내용들이 많으니 머리아파하지 않아도 된다.
일부 HTTP에 대한 이해가 없다면 어려울 수 있는 부분이 있는데 이럴 때는 기본설정을 사용하길 바란다.
전체적으로 기본 설정을 그대로 사용해도 서비스에는 전혀 지장이 없는 내용들이다.


.. toctree::
   :maxdepth: 2


.. _handling_http_requests_session_man:

세션관리
====================================

HTTP 클라이언트가 서버(STON)에 접속하면 HTTP 세션이 생성된다.
클라이언트는 HTTP 세션을 통해 서버에 저장된 여러 콘텐츠를 서비스 받는다. 
요청부터 응답까지를 하나의 **HTTP 트랜잭션** 이라고 부른다.
HTTP 세션은 여러 HTTP 트랜잭션을 순차적으로 처리한다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <ConnectionHeader>keep-alive</ConnectionHeader>
   <ClientKeepAliveSec>10</ClientKeepAliveSec>
   <KeepAliveHeader Max="0">ON</KeepAliveHeader>   
    
-  ``<ConnectionHeader> (기본: keep-alive)``    
   클라이언트에게 보내는 HTTP응답의 Connection헤더( ``keep-alive`` 또는 ``close`` )를 설정한다.
    

-  ``<ClientKeepAliveSec> (기본: 10초)``
   클라이언트 세션과 아무런 통신이 없는 상태로 설정된 시간이 경과하면 세션을 종료한다. 
   시간을 너무 길게 설정하면 통신을 하지 않는 세션이 지나치게 많아진다.
   너무 많은 세션을 유지하는 것만으로도 시스템엔 부하가 된다.

-  ``<KeepAliveHeader>``

    - ``ON (기본)`` HTTP응답에 Keep-Alive헤더를 명시한다.
      ``Max (기본: 0)`` 를 0보다 크게 설정하면 Keep-Alive헤더의 값으로 ``Max`` 값이 명시된다.
      이후 HTTP 트랜잭션이 발생할때마다 1씩 차감된다.
   
   - ``OFF`` HTTP응답에 Keep-Alive헤더를 생략한다.


HTTP 세션 유지정책
---------------------

STON은 가급적 Apache의 정책을 따른다.
특히 세션유지 정책은 HTTP헤더 값에 따른 변수가 많다.
HTTP 세션 유지정책에 영향을 주는 요소는 다음과 같다.

- 클라이언트 HTTP요청에 명시된 Connection헤더 ("ep-Alive" 또는 "Close")
- 가상호스트 ``<Connection>`` 설정
- 가상호스트 세션 Keep-Alive시간 설정
- 가상호스트 ``<Keep-Alive>`` 설정


1. 클라이언트 HTTP요청에 "Connection: Close"로 명시되어 있는 경우 ::

      GET / HTTP/1.1
      ...(생략)...
      Connection: Close
    
   이같은 HTTP요청에 대해서는 가상호스트 설정여부와 상관없이 
   "Connection: Close"로 응답한다. Keep-Alive헤더는 명시되지 않습니다. ::
   
      HTTP/1.1 200 OK
      ...(생략)...
      Connection: Close

   이 HTTP 트랜잭션이 완료되면 HTTP 연결을 종료한다.
   

2. ``<ConnectionHeader>`` 가 ``Close`` 로 설정된 경우 ::

      # server.xml - <Server><VHostDefault><Options>
      # vhosts.xml - <Vhosts><Vhost><Options>
      
      <ConnectionHeader>Close</ConnectionHeader>      
    
   클라이언트 HTTP요청과 상관없이 "Connection: Close"로 응답한다.
   Keep-Alive헤더는 명시되지 않는다. ::

      HTTP/1.1 200 OK
      ...(생략)...
      Connection: Close
      

3. ``<KeepAliveHeader>`` 가 ``OFF`` 로 설정된 경우 ::

      # server.xml - <Server><VHostDefault><Options>
      # vhosts.xml - <Vhosts><Vhost><Options>
      
      <ConnectionHeader>Keep-Alive</ConnectionHeader>
      <KeepAliveHeader>OFF</KeepAliveHeader>
    
   Keep-Alive헤더가 명시되지 않는다. HTTP 세션은 지속적으로 재사용가능하다. ::

      HTTP/1.1 200 OK
      ...(생략)...
      Connection: Keep-Alive


4. ``<KeepAliveHeader>`` 가 ``ON`` 으로 설정된 경우 ::

      # server.xml - <Server><VHostDefault><Options>
      # vhosts.xml - <Vhosts><Vhost><Options>
      
      <ConnectionHeader>Keep-Alive</ConnectionHeader>
      <ClientKeepAliveSec>10</ClientKeepAliveSec>
      <KeepAliveHeader>ON</KeepAliveHeader>      
    
   Keep-Alive헤더가 명시된다.
   timeout값은 세션 Keep-Alive시간 설정을 사용한다. ::
    
      HTTP/1.1 200 OK
      ...(생략)...
      Connection: Keep-Alive
      Keep-Alive: timeout=10

   .. note::

      < ``<Keep-Alive>`` 와 ``<ClientKeepAliveSec>`` 의 관계 >
    
      ``<Keep-Alive>`` 설정시 ``<ClientKeepAliveSec>`` 를 참고하지만 ``<ClientKeepAliveSec>`` 는 보다 근본적인 문제와 관련이 있다. 
      성능이나 자원적으로 가장 중요한 이슈는 Idle세션(=HTTP 트랜잭션이 발생되지 않는 세션)의 정리시점을 잡는 것이다.
      HTTP 헤더 설정은 동적으로 변경되거나 때로 생략될 수 있지만 Idle세션 정리는 훨씬 민감한 문제이다. 
      이런 이유 때문에 ``<ClientKeepAliveSec>`` 는 ``<KeepAliveHeader>`` 에 통합되지 않고 별도로 존재한다.
   

5. ``<KeepAliveHeader>`` 의 ``Max`` 속성이 설정된 경우 ::

      # server.xml - <Server><VHostDefault><Options>
      # vhosts.xml - <Vhosts><Vhost><Options>
      
      <ConnectionHeader>Keep-Alive</ConnectionHeader>
      <ClientKeepAliveSec>10</ClientKeepAliveSec>
      <KeepAliveHeader Max="50">ON</KeepAliveHeader>      
    
   Keep-Alive헤더에 max값을 명시한다. 
   이 세션은 max회만큼 사용이 가능하며 HTTP 트랜잭션이 진행될때마다 1씩 감소된다. ::
    
      HTTP/1.1 200 OK
      ...(생략)...
      Connection: Keep-Alive
      Keep-Alive: timeout=10, max=50


6. Keep-Alive의 max가 만료된 경우 ::

   위의 설정대로 max가 설정되었다면 max는 점차 줄어 다음처럼 1까지 도달하게 된다. ::

      HTTP/1.1 200 OK
      ...(생략)...
      Connection: Keep-Alive
      Keep-Alive: timeout=10, max=1
    
   이 응답은 현재 세션으로 앞으로 1번 HTTP 트랜잭션진행이 가능하다는 의미이다. 
   이 세션으로 HTTP 요청이 한번 더 진행될 경우 다음과 같이 "Connection: Close"로 응답한다. ::
    
      HTTP/1.1 200 OK
      ...(생략)...
      Connection: Close    



클라이언트 Cache-Control
====================================

클라이언트 Cache-Control과 관련된 설정을 다룬다.

Age 헤더
---------------------

Age헤더는 캐싱된 순간부터 경과시간(초)을 의미하며 
`RFC2616 - 13.2.3 Age Calculations <http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.2.3>`_ 에 의하여 계산된다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <AgeHeader>OFF</AgeHeader>   
    
-  ``<AgeHeader>``
    
   -  ``OFF (기본)`` Age헤더를 생략한다.
   
   -  ``OFF`` Age헤더를 명시한다.


Expires 헤더
---------------------

Expires헤더를 재설정한다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <RefreshExpiresHeader Base="Access">OFF</RefreshExpiresHeader>   
    
-  ``<RefreshExpiresHeader>``
    
   -  ``OFF (기본)`` 원본서버에서 응답한 Expires헤더를 클라이언트에게 명시한다.
      원본서버에서 Expires헤더가 생략되었다면 클라이언트 응답에도 Expires헤더가 생략된다.
   
   -  ``ON``  Expires조건을 반영하여 Expires헤더를 명시한다.
      조건에 해당하지 않는 콘텐츠는 ``OFF`` 설정과 동일하게 동작한다.
   
Expires조건은 Apache의 `mod_expires <http://httpd.apache.org/docs/2.2/mod/mod_expires.html>`_ 와 동일하게 동작한다. 
특정 조건(URL이나 MIME Type)에 해당하는 콘텐츠의 Expires헤더와 Cache-Control 값을 설정할 수 있다. 
Cache-Control의 max-age값은 설정된 Expires시간에서 요청한 시간을 뺀 값이 된다. 

Expires조건은 /svc/{가상호스트 이름}/expires.txt에 설정한다. ::

   # /svc/www.exmaple.com/expires.txt
   # 구분자는 콤마(,)이며 {조건},{시간},{기준} 순서로 표기한다.

   $URL[/test.jpg], 86400
   /test.jpg, 86400
   *, 86400, access
   /test/1.gif, 60 sec
   /test/*.dat, 30 min, modification
   $MIME[application/shockwave], 1 years
   $MIME[application/octet-stream], 7 weeks, modification
   $MIME[image/gif], 3600, modification

-  **조건**

   URL과 MIME Type 2가지로 설정이 가능하다. 
   URL일 경우 $URL[...]로, MIME Type일 경우 $MIME[...]로 표기한다. 
   패턴표현이 가능하며 $표현이 생략된 경우 URL로 인식한다.

-  **시간**

   Expires만료시간을 설정한다. 
   시간단위 표현을 지원하며 단위를 명시하지 않을 경우 초로 계산된다.

-  **기준**

   Expires만료시간의 기준시점을 설정한다. 
   별도로 기준시점을 명시하지 않으면 Access가 기준시점으로 명시된다. 
   Access는 현재 시간을 기준으로 한다. 
   다음은 MIME Type이 image/gif인 파일에 대하여 접근시간으로부터 
   1일 12시간 후로 Expires헤더 값을 설정하는 예제이다. ::
    
      $MIME[image/gif], 1 day 12 hours, access
      
   Modification은 원본서버에서 보낸 Last-Modified를 기준으로 한다. 
   다음은 모든 jpg파일에 대하여 Last-Modified로부터 30분 뒤를 
   Expires값으로 설정하는 예제이다. ::
    
      *.jpg, 30min, modification
        
   Modification의 경우 계산된 Expires값이 현재시간보다 과거의 시간일 경우 현재시간을 명시한다.
   만약 원본서버에서 Last-Modified헤더를 제공하지 않는다면 Expires헤더를 보내지 않는다.


ETag 헤더
---------------------

클라이언트에게 보내는 HTTP응답에 ETag 헤더 명시여부를 설정한다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <ETagHeader>ON</ETagHeader>   
    
-  ``<ETagHeader>``
    
   -  ``ON (기본)`` ETag헤더를 명시한다.
   
   -  ``OFF``  ETag헤더를 생략한다.
   
   


응답 헤더
====================================

HTTP 요청/응답 헤더 변경
---------------------

클라이언트 HTTP요청과 응답을 특정 조건에 따라 변경한다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <ModifyHeader FirstOnly="OFF">OFF</ModifyHeader>   
    
-  ``<ModifyHeader>``
    
   -  ``OFF (기본)`` 변경하지 않는다.
   
   -  ``ON`` 헤더 변경조건에 따라 헤더를 변경한다.
   
헤더 변경시점을 정확히 이해하자.

-  **HTTP 요청헤더 변경시점**

   클라이언트 HTTP 요청을 최초로 인식하는 시점에 헤더를 변경한다. 
   헤더가 변경되었다면 변경된 상태로 Cache 모듈에서 처리된다.
   단, Host헤더와 URI는 변조할 수 없다.

-  **HTTP 응답헤더 변경시점**

   클라이언트 응답 직전에 헤더를 변경한다. 
   단, Content-Length는 변경할 수 없다.   

      
헤더 변경조건은 /svc/{가상호스트 이름}/headers.txt에 설정한다. 
헤더는 멀티로 설정이 가능하므로 조건과 일치한다면 모든 변경설정이 동시에 적용된다. 

최초 조건에만 변경을 원할 경우 ``FirstOnly`` 속성을 ``ON`` 으로 설정한다.
서로 다른 조건이 같은 헤더를 변경하는 경우 Last-Win이 되거나 명시적으로 Append할 수 있다. ::

   # /svc/www.example.com/headers.txt
   # 구분자는 콤마(,)이다.
   
   # 요청변경
   # {Match}, {$REQ}, {Action(set|unset|append)} 순서로 표기한다.
   $IP[192.168.1.1], $REQ[SOAPAction], unset
   $IP[192.168.2.1-255], $REQ[accept-encoding: gzip], set
   $IP[192.168.3.0/24], $REQ[cache-control: no-cache], append
   $IP[192.168.4.0/255.255.255.0], $REQ[x-custom-header], unset
   $IP[AP], $REQ[X-Forwarded-For], unset
   $HEADER[user-agent: *IE6*], $REQ[accept-encoding], unset
   $HEADER[via], $REQ[via], unset
   $URL[/source/*.zip], $REQ[accept-encoding: deflate], set
   
   # 응답변경
   # {Match}, {$RES}, {Action(set|unset|append)}, {condition} 순서로 표기한다.
   # {condition}은 특정 응답코드에 한하여 헤더를 변경할 수 있지만 필수는 아니다.
   $IP[192.168.1.1], $RES[via: STON for CDN], set
   $IP[192.168.2.1-255], $RES[X-Cache], unset, 200
   $IP[192.168.3.0/24], $RES[cache-control: no-cache, private], append, 3xx
   $IP[192.168.4.0/255.255.255.0], $RES[x-custom-header], unset
   $HEADER[user-agent: *IE6*], $RES[vary], unset
   $HEADER[x-custom-header], $RES[cache-control: no-cache, private], append, 5xx
   $URL[/source/*], $RES[cache-control: no-cache], set, 404
   /secure/*.dat, $RES[x-custom], unset, 200
    
{Match}는 IP, GeoIP, Header, URL 4가지로 설정이 가능하다.

-  **IP**
   $IP[...]로 표기하며 IP, IP Range, Bitmask, Subnet 네 가지 형식을 지원한다.

-  **GeoIP**
   $IP[...]로 표기하며 반드시 :ref:`access-control-geoip` 가 설정되어 있어야 한다.
   국가코드는 `ISO 3166-1 alpha-2 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2>`_ 와 `ISO 3166-1 alpha-3 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3>`_ 를 지원한다.
     
-  **Header**
   $HEADER[Key : Value]로 표기한다. 
   Value는 명확한 표현과 패턴을 지원한다. 
   Value가 생략된 경우에는 Key에 해당하는 헤더의 존재유무를 조건으로 판단한다.
    
-  **URL**
   $URL[...]로 표기하며 생략이 가능하다. 명확한 표현과 패턴을 인식한다.
    
{$REQ}와 {$RES}는 헤더변경 방법을 설정한다.
일반적으로 ``set`` 과 ``append`` 의 경우 {Key: Value}로 설정하며, 
Value가 입력되지 않은 경우 빈 값("")이 입력된다. 
``unset`` 의 경우 {Key}만 입력한다.

{Action}은 ``set`` , ``unset`` , ``append`` 3가지로 설정이 가능하다.

-  ``set``  요청/응답 헤더에 설정되어 있는 Key와 Value를 헤더에 추가한다. 
   이미 같은 Key가 존재한다면 이전 값을 덮어쓴다.    

-  ``unset`` 요청/응답 헤더에 설정되어 있는 Key에 해당하는 헤더를 삭제한다.

-  ``append``  ``set`` 과 유사하나 해당 Key가 존재한다면 기존의 Value와 설정된 Value사이에 Comma(,)로 구분하여 값을 결합한다.

{Condition}은 200이나 304같은 구체적인 응답 코드외에 2xx, 3xx, 4xx, 5xx처럼 응답코드 계열조건으로 설정한다. 
{Match}와 일치하더라도 {Condition}과 일치하지 않는다면 변경이 반영되지 않는다.
{Condition}이 생략된 경우 응답코드를 검사하지 않는다.


원본 헤더
---------------------

성능과 보안상의 이유로 원본서버가 보내는 헤더 중 표준헤더만을 선택적으로 인식한다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <OriginalHeader>OFF</OriginalHeader>   
    
-  ``<OriginalHeader>``

   -  ``OFF (기본)`` 표준헤더가 아니라면 무시한다. 
   
   -  ``ON`` cookie, set-cookie, set-cookie2를 제외한 모든 헤더를 저장하여 클라이언트에게 전달한다.
      단, 메모리와 저장비용을 좀 더 소비한다.


Via 헤더
---------------------

클라이언트에게 보내는 HTTP응답에 Via 헤더 명시여부를 설정한다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <ViaHeader>ON</ViaHeader>   
    
-  ``<ViaHeader>``
    
   - ``ON (기본)`` Via헤더를 다음과 같이 명시한다.
     ::
      
        Via: STON/2.0.0
   
   - ``OFF``  Via헤더를 생략한다.
   
   
Server 헤더
---------------------
 
클라이언트에게 보내는 HTTP응답에 Server 헤더 명시여부를 설정한다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <ServerHeader>ON</ServerHeader>   
    
-  ``<ServerHeader>``
    
   -  ``ON (기본)`` 원본서버의 Server헤더를 명시한다. ::
   
   -  ``OFF``  Server헤더를 생략한다.



URL 전처리
====================================

`정규표현식 <http://en.wikipedia.org/wiki/Regular_expression>`_ 을 사용하여 요청된 URL을 변경한다. 
URL전처리가 설정되어 있다면 모든 클라이언 요청(HTTP 또는 File I/O)은 반드시 URL Rewriter를 거친다.

.. figure:: img/urlrewrite1.png
   :align: center
      
   URL Rewriter를 통과해야 가상호스트에 갈 수 있다.
   
만약 URL Rewriter에 의해 접근하려는 Host이름이 변경되었다면 클라이언트 HTTP요청의 Host헤더가 변경된 것으로 간주한다.
URL 전처리는 가상호스트 설정(vhosts.xml)에 설정한다.
대부분의 설정이 가상호스트에 종속되지만, URL전처리의 경우 클라이언트가 요청한 Host의 이름을 변경할 수 있으므로 가상호스트와 같은 레벨로 설정한다. ::

   # vhosts.xml
   
   <Vhosts>
      <Vhost ...> ... </Vhost>
      <Vhost ...> ... </Vhost>
      <URLRewrite ...> ... </URLRewrite>
      <URLRewrite ...> ... </URLRewrite>
   </Vhosts>
    
멀티로 설정할 수 있으며 순차적으로 정규표현식 일치 여부를 비교한다. ::

   # vhosts.xml - <Vhosts>
   
   <URLRewrite AccessLog="Replace">
       <Pattern>www.exmaple.com/([^/]+)/(.*)</Pattern>
       <Replace>#1.exmaple.com/#2</Replace>
   </URLRewrite>
    
-  ``<URLRewrite>``

   URL전처리를 설정한다.
   ``AccessLog (기본: Replace)`` 속성은  Access로그에 기록될 URL을 설정한다. 
   ``Replace`` 인 경우 변환 후 URL(/logo.jpg)을, ``Pattern`` 인 경우 변환 전 
   URL(/baseball/logo.jpg)을 Access로그에 기록한다.
   
   -  ``<Pattern>`` 매칭시킬 패턴을 설정한다. 
      한개의 패턴은 ( ) 괄호를 사용하여 표현된다.
   
   -  ``<Replace>`` 변환형식을 설정한다. 
      일치된 패턴에 대해서는 #1, #2와 같이 사용할 수 있다. #0는 요청 URL전체를 의미한다. 
      패턴은 최대 9개(#9)까지 지정할 수 있다.
      
처리량은 :ref:`monitoring_stats` 로 제공되며 :ref:`api-graph-urlrewrite` 으로도 확인할 수 있다. 
URL전처리는 :ref:`media-trimming` , :ref:`media-hls` 등 다른 기능들과 결합하여 표현을 간결하게 만든다. ::

   # vhosts.xml - <Vhosts>

   <URLRewrite>
       <Pattern>example.com/([^/]+)/(.*)</Pattern>
       <Replace>example.com/#1.php?id=#2</Replace>
   </URLRewrite>
   // Pattern : example.com/releasenotes/1.3.4
   // Replace : example.com/releasenotes.php?id=1.3.4

   <URLRewrite>
       <Pattern>example.com/download/(.*)</Pattern>
       <Replace>download.example.com/#1</Replace>
   </URLRewrite>
   // Pattern : example.com/download/1.3.4
   // Replace : download.example.com/1.3.4

   <URLRewrite>
       <Pattern>example.com/img/(.*\.(jpg|png).*)</Pattern>
       <Replace>example.com/#1/STON/composite/watermark1</Replace>
   </URLRewrite>
   // Pattern : example.com/img/image.jpg?date=20140326
   // Replace : example.com/image.jpg?date=20140326/STON/composite/watermark1

   <URLRewrite>
       <Pattern>example.com/preview/(.*)\.(mp3|mp4|m4a)$</Pattern>
       <Replace><![CDATA[example.com/#1.#2?&end=30&boost=10&bandwidth=2000&ratio=100]]></Replace>
   </URLRewrite>
   // Pattern : example.com/preview/audio.m4a
   // Replace : example.com/audio.m4a?end=30&boost=10&bandwidth=2000&ratio=100

   <URLRewrite>
       <Pattern>example.com/(.*)\.mp4\.m3u8$</Pattern>
       <Replace>example.com/#1.mp4/mp4hls/index.m3u8</Replace>
   </URLRewrite>
   // Pattern : example.com/video.mp4.m3u8
   // Replace : example.com/video.mp4/mp4hls/index.m3u8

   <URLRewrite>
       <Pattern>example.com/(.*)_(.*)_(.*)</Pattern>
       <Replace>example.com/#0/#1/#2/#3</Replace>
   </URLRewrite>
   // Pattern : example.com/video.mp4_10_20
   // Replace : example.com/example.com/video.mp4_10_20/video.mp4/10/20
    
패턴표현에 XML의 5가지 특수문자( " & ' < > )가 들어갈 경우 반드시 <![CDATA[ ... ]]>로 묶어주어야 올바르게 설정된다. 
:ref:`wm` 을 통해 설정할 때 모든 패턴은 CDATA로 처리된다.