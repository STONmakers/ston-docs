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
- 가상호스트 Connection헤더 설정
- 가상호스트 세션 Keep-Alive시간 설정
- 가상호스트 ``Keep-Alive``헤더 설정


1. **클라이언트 HTTP요청에 "Connection: Close"로 명시되어 있는 경우** ::

    GET / HTTP/1.1
    ...(생략)...
    Connection: Close
    
   이같은 HTTP요청에 대해서는 가상호스트 설정여부와 상관없이 
   "Connection: Close"로 응답한다. Keep-Alive헤더는 명시되지 않습니다. ::

    HTTP/1.1 200 OK
    ...(생략)...
    Connection: Close

   이 HTTP 트랜잭션이 완료되면 HTTP 연결을 종료한다.
   

#. **가상호스트 ConnectionHeader가 "Close"으로 설정된 경우** ::

    <Options>
        <ConnectionHeader>Close</ConnectionHeader>
    </Options>
    
   클라이언트 HTTP요청과 상관없이 "Connection: Close"로 응답한다.
   Keep-Alive헤더는 명시되지 않는다. ::

    HTTP/1.1 200 OK
    ...(생략)...
    Connection: Close


#. **가상호스트 KeepAliveHeader가 OFF로 설정된 경우** ::

    <Options>
        <ConnectionHeader>Keep-Alive</ConnectionHeader>
        <KeepAliveHeader>OFF</KeepAliveHeader>
    </Options>
    
   Keep-Alive헤더가 명시되지 않는다. HTTP세션은 지속적으로 재사용가능하다. ::

   HTTP/1.1 200 OK
   ...(생략)...
   Connection: Keep-Alive

#. **가상호스트 KeepAliveHeader가 ON으로 설정된 경우** ::

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

   ** ``<Keep-Alive>``와 ``<ClientKeepAliveSec>`` 관계**
    
   ``<Keep-Alive>`` 설정시 ``<ClientKeepAliveSec>`` 를 참고하지만 ``<ClientKeepAliveSec>`` 는 
   보다 근본적인 문제와 관련이 있다. 
   성능이나 자원적으로 가장 중요한 이슈는 
   "Idle세션(=HTTP 트랜잭션이 발생되지 않는 세션)을 언제 정리할 것인가?" 이다. 
   HTTP 헤더 설정은 동적으로 변경되거나 때로 생략될 수 있지만 
   Idle세션 정리는 훨씬 민감한 문제이다. 
   이런 이유 때문에 ``<ClientKeepAliveSec>`` 는 ``<KeepAliveHeader>`` 에 통합되지 않고 별도로 존재한다.
   

#. **가상호스트 ``<KeepAliveHeader>`` 의 Max속성이 설정된 경우** ::

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


#. **Keep-Alive의 max가 만료된 경우** ::

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
