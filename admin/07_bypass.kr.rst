.. _bypass:

바이패스
******************

개인화된 웹 페이지처럼 Caching하면 안되는 클라이언트 요청이 있다.
바이패스는 HTTP트랜잭션 단위로 동작한다.


.. toctree::
   :maxdepth: 2

바이패스 조건
====================================

바이패스는 Caching정책보다 우선한다.

No-Cache 요청 바이패스
-----------------------

클라이언트가 no-cache요청을 바이패스한다. ::

    GET / HTTP/1.1
    ...
    cache-control: no-cache 또는 cache-control:max-age=0
    pragma: no-cache
    
::

    <Options>
         <BypassNoCacheRequest>OFF</BypassNoCacheRequest>
    </Options>
    
-  ``<BypassNoCacheRequest>``

   - ``OFF (기본)`` Cache모듈이 처리한다.
   
   - ``ON`` 원본서버로 바이패스한다.
   
.. note::

    이 설정은 클라이언트 동작(아마도 ``ctrl`` + ``F5`` )에 의해 판단된다.
    그러므로 대량의 바이패스가 원본에 부담을 줄 수 있다.
   
 
GET/POST 바이패스
-----------------------    

바이패스가 GET/POST요청의 기본동작이 되도록 설정할 수 있다. ::

    <Options>
        <BypassPostRequest>ON</BypassPostRequest>
        <BypassGetRequest>OFF</BypassGetRequest>
    </Options>
    
-  ``<BypassPostRequest>``

   - ``ON (기본)`` POST요청을 원본서버로 바이패스한다.
   
   - ``OFF`` POST요청을 STON이 처리한다.
   
-  ``<BypassGetRequest>``

   - ``OFF (기본)`` GET요청을 STON이 처리한다.

   - ``ON`` GET요청을 원본서버로 바이패스한다.
   
**가상호스트 ACL** 과 동일한 조건을 모두 지원한다.
바이패스 예외조건은 /svc/{가상호스트 이름}/bypass.txt 에 설정한다. ::

    # /svc/www.example.com/bypass.txt
    $IP[192.168.2.1-255]
    /index.html   
    
cache나 bypass조건을 명확하게 명시하지 않은 경우 기본설정과 반대로 동작한다.
예를 들어 기본조건이 바이패스라면 예외조건의 기본조건은 Caching이다.
2번째 파라미터를 사용하면 보다 분명하게 조건을 설정할 수 있다. ::

    # /svc/www.winesoft.co.kr/bypass.txt
    $HEADER[cookie: *ILLEGAL*], cache               // 항상 Caching처리
    !HEADER[referer:]                               // 기본 설정에 따라 다름
    !HEADER[referer] & !HEADER[user-agent], bypass  // 항상 바이패스
    $URL[/source/public.zip]                        // 기본 설정에 따라 다름

정리하면 우선순위는 다음과 같다.

1. No-Cache 바이패스
2. bypass.txt에 bypass라고 명시된 경우
3. bypass.txt의 기본 설정


바이패스 동작
====================================

바이패스의 구체적인 동작방식을 설정한다.

원본 고정
-----------------------

같은 클라이언트가 항상 동일한 서버로 바이패스되도록 설정한다. ::

     <Options>
        <BypassPostRequest OriginAffinity="ON">...</BypassPostRequest>
        <BypassGetRequest OriginAffinity="ON">...</BypassGetRequest>
    </Options>
    
-  ``OriginAffinity``

   - ``ON (기본)`` 클라이언트 요청이 항상 같은 서버로 바이패스되는 것을 보장한다. 
     단, 같은 소켓임을 보장하지는 않는다. 
     바이패스해야 하는 원본서버와 연결된 모든 소켓이 끊어지는 상황이 발생할 수도 있다. 
     하지만 이런 경우에도 해당 서버로 새로운 소켓연결을 요청한다.
     
     .. figure:: img/private_bypass3.jpg
        :align: center
      
        항상 같은서버로 바이패스된다.
     
     바이패스하던 원본서버가 장애로 배제되거나 DNS에서 빠질 경우 새로운 서버로 
     바이패스된다.
   
   - ``OFF`` 원본서버로 바이패스한다.


