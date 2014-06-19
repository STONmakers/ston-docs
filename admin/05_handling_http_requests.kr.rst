.. _handling_http_requests:

HTTP 클라이언트
******************

HTTP 클라이언트와 STON사이의 HTTP 통신에 대해 설정한다.

.. toctree::
   :maxdepth: 2


HTTP 세션
====================================

HTTP클라이언트가 서버(STON)에 접속하면 HTTP세션이 생성된다.
클라이언트는 HTTP 세션을 통해 서버에 저장된 여러 콘텐츠를 서비스 받는다. 
요청부터 응답까지를 하나의 **HTTP 트랜잭션** 이라고 부른다.
HTTP세션은 여러 HTTP트랜잭션을 순차적으로 처리한다.

다음은 HTTP세션과 관련된 설정이다. ::

   <Options>
      <ConnectionHeader>keep-alive</ConnectionHeader>
      <ClientKeepAliveSec>10</ClientKeepAliveSec>
      <KeepAliveHeader Max="0">ON</KeepAliveHeader>
   </Options>
    
-  ``<ConnectionHeader> (기본: keep-alive)``
    
   클라이언트에게 보내는 HTTP응답의 Connection헤더( ``keep-alive`` 또는 ``close`` )를 설정한다.
    

-  ``<ClientKeepAliveSec> (기본: 10초)``

   클라이언트 세션과 아무런 통신이 없는 상태로 설정된 시간이 경과하면 세션을 종료한다. 
   시간을 너무 길게 설정하면 통신을 하지 않는 세션이 지나치게 많아진다.
   너무 많은 세션은 성능저하의 원인이 된다.

-  ``<KeepAliveHeader>``

    - ``ON (기본)`` HTTP응답에 Keep-Alive헤더를 명시한다.
      ``Max (기본: 0)`` 를 0보다 크게 설정하면 Keep-Alive헤더의 
      값으로 max가 명시된다.
      이후 HTTP 트랜잭션이 발생할때마다 1씩 차감된다.
   
   - ``OFF`` HTTP응답에 Keep-Alive헤더를 생략한다.

HTTP세션 유지정책
---------------------

STON은 Apache의 정책을 준수한다. 
여기서는 판단 우선순위에 따라 기술한다. 
HTTP세션 유지정책에 영향을 주는 요소는 다음과 같다.

- 클라이언트 HTTP요청에 명시된 Connection헤더 ("ep-Alive" 또는 "Close")
- 가상호스트 ``<Connection>`` 설정
- 가상호스트 세션 Keep-Alive시간 설정
- 가상호스트 ``<Keep-Alive>`` 설정


**1. 클라이언트 HTTP요청에 "Connection: Close"로 명시되어 있는 경우** ::

   GET / HTTP/1.1
   ...(생략)...
   Connection: Close
    
이같은 HTTP요청에 대해서는 가상호스트 설정여부와 상관없이 
"Connection: Close"로 응답한다. Keep-Alive헤더는 명시되지 않습니다. ::
   
   HTTP/1.1 200 OK
   ...(생략)...
   Connection: Close

이 HTTP 트랜잭션이 완료되면 HTTP 연결을 종료한다.
   

**2.**  ``<ConnectionHeader>`` **가** ``Close`` **로 설정된 경우** ::

   <Options>
      <ConnectionHeader>Close</ConnectionHeader>
   </Options>
    
클라이언트 HTTP요청과 상관없이 "Connection: Close"로 응답한다.
Keep-Alive헤더는 명시되지 않는다. ::

    HTTP/1.1 200 OK
    ...(생략)...
    Connection: Close


**3.**  ``<KeepAliveHeader>`` **가** ``OFF`` **로 설정된 경우** ::

    <Options>
        <ConnectionHeader>Keep-Alive</ConnectionHeader>
        <KeepAliveHeader>OFF</KeepAliveHeader>
    </Options>
    
Keep-Alive헤더가 명시되지 않는다. HTTP세션은 지속적으로 재사용가능하다. ::

   HTTP/1.1 200 OK
   ...(생략)...
   Connection: Keep-Alive


**4.**  ``<KeepAliveHeader>`` **가** ``ON`` **으로 설정된 경우** ::

   <Options>
      <ConnectionHeader>Keep-Alive</ConnectionHeader>
      <ClientKeepAliveSec>10</ClientKeepAliveSec>
      <KeepAliveHeader>ON</KeepAliveHeader>
   </Options>
    
Keep-Alive헤더가 명시된다.
timeout값은 세션 Keep-Alive시간 설정을 사용한다. ::
    
    HTTP/1.1 200 OK
    ...(생략)...
    Connection: Keep-Alive
    Keep-Alive: timeout=10

.. note::

   ** ``<Keep-Alive>`` 와 ``<ClientKeepAliveSec>`` 의 관계**
    
   ``<Keep-Alive>`` 설정시 ``<ClientKeepAliveSec>`` 를 참고하지만 ``<ClientKeepAliveSec>`` 는 
   보다 근본적인 문제와 관련이 있다. 
   성능이나 자원적으로 가장 중요한 이슈는 
   `` "Idle세션(=HTTP 트랜잭션이 발생되지 않는 세션)을 언제 정리할 것인가?" `` 이다. 
   HTTP 헤더 설정은 동적으로 변경되거나 때로 생략될 수 있지만 
   Idle세션 정리는 훨씬 민감한 문제이다. 
   이런 이유 때문에 ``<ClientKeepAliveSec>`` 는 ``<KeepAliveHeader>`` 에 통합되지 않고 별도로 존재한다.
   

**5.**  ``<KeepAliveHeader>`` **의** ``Max`` **속성이 설정된 경우** ::

    <Options>
       <ConnectionHeader>Keep-Alive</ConnectionHeader>
       <ClientKeepAliveSec>10</ClientKeepAliveSec>
       <KeepAliveHeader Max="50">ON</KeepAliveHeader>
    </Options>
    
Keep-Alive헤더에 max값을 명시한다. 
이 세션은 max회만큼 사용이 가능하며 HTTP 트랜잭션이 진행될때마다 1씩 감소된다. ::
    
    HTTP/1.1 200 OK
    ...(생략)...
    Connection: Keep-Alive
    Keep-Alive: timeout=10, max=50


**6. Keep-Alive의 max가 만료된 경우** ::

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

    <Options>
       <AgeHeader>OFF</AgeHeader>
    </Options>
    
-  ``<AgeHeader>``
    
   -  ``OFF (기본)`` Age헤더를 생략한다.
   
   -  ``OFF`` Age헤더를 명시한다.

Expires 헤더
---------------------

Expires헤더를 재설정한다. ::

    <Options>
       <RefreshExpiresHeader Base="Access">OFF</RefreshExpiresHeader>
    </Options>
    
-  ``<RefreshExpiresHeader>``
    
   -  ``OFF (기본)`` 원본서버에서 응답한 Expires헤더를 클라이언트에게 명시한다.
      원본서버에서 Expires헤더가 생략되었다면 클라이언트 응답에도 Expires헤더가 생략된다.
   
   -  ``ON``  Expires조건을 반영하여 Expires헤더를 명시한다.
      조건에 해당하지 않는 콘텐츠는 ``OFF`` 설정과 동일하게 동작한다.
   
Expires조건은 Apache의 `mod_expires <http://httpd.apache.org/docs/2.2/mod/mod_expires.html>`_ 
와 동일하게 동작한다. 특정 조건(URL이나 MIME Type)에 해당하는 콘텐츠의 
Expires헤더와 Cache-Control 값을 설정할 수 있다. 
Cache-Control의 max-age값은 설정된 Expires시간에서 요청한 시간을 뺀 값이 된다. 

Expires조건은 /svc/{가상호스트 이름}/expires.txt에 설정한다. ::

   # /svc/www.winesoft.co.kr/expires.txt
   # 구분자는 콤마(,)이며 {조건},{시간},{기준} 순서로 표기한다.

   $URL[/test.jpg], 86400
   /test.jpg, 86400
   *, 86400, access
   /test/1.gif, 60 sec
   /test/*.dat, 30 min, modification
   $MIME[application/shockwave], 1 years
   $MIME[application/octet-stream], 7 weeks, modification
   $MIME[image/gif], 3600, modification

- **조건**

    URL과 MIME Type 2가지로 설정이 가능하다. 
    URL일 경우 $URL[...]로, MIME Type일 경우 $MIME[...]로 표기한다. 
    패턴표현이 가능하며 $표현이 생략된 경우 URL로 인식한다.

- **시간**

    Expires만료시간을 설정한다. 
    시간단위 표현을 지원하며 단위를 명시하지 않을 경우 초로 계산된다.

- **기준**

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
        
    Modification의 경우 계산된 Expires값이 현재시간보다 과거의 시간일 경우 
    현재시간을 명시한다. 
    만약 원본서버에서 Last-Modified헤더를 제공하지 않는다면 Expires헤더를 
    보내지 않는다.
