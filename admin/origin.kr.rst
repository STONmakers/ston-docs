.. _origin:

7장. 원본서버
******************

이 장에서는 STON과 원본서버의 관계에 대해 설명한다.
원본서버란 일반적으로 HTTP 규격을 준수하는 웹서버를 의미한다.
관리자라면 원본을 보호하기 위해 이번 장의 모든 내용을 숙지할 필요가 있다.
이를 바탕으로 원본장애에도 내구성을 갖춘 유연한 서비스를 구축할 수 있다.

원본서버는 보호되어야 한다.
장애의 종류가 다양한만큼 대처방안도 다양하다.
원본보호 정책을 적절히 구성하면 여유로운 점검시간을 가질 수 있다.


.. toctree::
   :maxdepth: 2


장애감지와 자동복구
====================================

원본서버에 장애가 발생하면 자동배제한다.
다시 안정화됐다고 판단하면 서비스에 투입한다. ::

    <OriginOptions>
        <ConnectTimeout>3</ConnectTimeout>
        <ReceiveTimeout>10</ReceiveTimeout>
        <Exclusion>3</Exclusion>
        <Recovery Cycle="10">5</Recovery>
    </OriginOptions>

-  ``<ConnectTimeout> (기본: 3초)``
    n초 이내에 원본서버와 접속이 이루어지지 않는 경우 접속실패로 간주한다.
   
-  ``<ReceiveTimeout> (기본: 10초)``
    정상적인 HTTP요청에도 불구하고 원본서버가 HTTP응답을 n초 동안 보내지 않는 경우 전송실패로 간주한다.   

-  ``<Exclusion> (기본: 3회)``
    원본서버에서 연속적으로 n번 장애상황( ``<ConnectTimeout>`` 또는 ``<ReceiveTimeout>`` )이 발생하면 해당 서버를 유효 원본서버 목록에서 배제한다. 
    배제 전 정상적인 통신이 이루어진다면 이 값은 다시 0으로 초기화된다.

-  ``<Recovery> (기본: 5회, Cycle: 10초)``
    배제된 원본서버를 다시 서비스에 투입하기 위하여 설정된 ``Cycle`` 마다 다음과 같이 요청한다. ::
    
       GET / HTTP/1.1
      
    원본서버로 연결이 되고 (응답코드 여부와 상관없이)응답이 n회 되면 서비스에 재투입한다. 
    
    
장애/복구 초기화
====================================

API를 통해 가상호스트의 원본서버 배제/복구를 초기화한다. 
또한 현재 사용 중인 세션을 재사용하지 않고 새롭게 연결을 생성한다. ::

   http://127.0.0.1:10040/command/resetorigin       // 모든 가상호스트
   http://127.0.0.1:10040/command/resetorigin?vhost=www.example.com



DNS 백업
====================================

원본서버 주소가 Domain으로 설정되어 있다면 항상 최신의 Resolving결과가 사용된다.
하지만 DNS서버의 장애 또는 네트워크 장애로 인하여 Resolving이 실패할 경우 마지막으로 Resolving된 IP에 부하가 집중될 수 있다. ::

   <OriginOptions>
      <DNSBackup>5min</DNSBackup>
   </OriginOptions>
    
-  ``<DNSBackup> (기본: 5min)``
   Resolving장애 상황이 발생하면 최근(일정시간 동안) 사용된 Unique한 IP들을 모두 사용하여 부하를 분산한다.


.. _origin-busysessioncount:

과부하 판단
====================================

처음 요청되는 콘텐츠는 항상 원본서버에 요청해야 한다.
하지만 이미 Caching된 콘텐츠라면 좀 더 유연하게 대처할 수 있다.
원본서버가 과부하 상태라고 판단되면 갱신을 늦추어 원본부하를 높이지 않는다. ::

   <OriginOptions>
      <BusySessionCount>100</BusySessionCount>
   </OriginOptions>

-  ``<BusySessionCount> (기본: 100개)``
   원본서버와 HTTP트랜잭션을 진행 중인 세션 수가 일정개수를 넘으면 과부하 상태로 판단한다.
   과부하 상태에서 만료된 컨텐츠를 갱신하기 위해 원본서버로 접속하지 않도록 TTL을 :ref:`caching-policy-ttl-originbusy` 만큼 연장한다.
   무조건 원본서버로 요청이 가도록 하려면 이 값을 아주 크게 설정하면 된다.
   

.. _origin-balancemode:

원본 선택
====================================

원본서버 주소가 멀티(2개 이상)로 구성되어 있을 때 원본서버 선택정책을 설정한다. ::

   <OriginOptions>
      <BalanceMode>RoundRobin</BalanceMode>
   </OriginOptions>

-  ``<BalanceMode> (기본: RoundRobin)``

   -  ``RoundRobin (기본)`` 
      모든 원본서버가 균등하게 요청을 받도록 Round-Robin으로 동작한다. 
      연결된 Idle세션은 해당 서버로 요청이 필요할 때만 사용한다.
   
   -  ``Session``      
      재사용할 수 있는 세션이 있다면 우선 사용한다. 
      신규 세션이 필요하면 Round-Robin으로 할당한다.
      
=========== =================================================================== =====================================================
/           RoundRobin                                                          Session
=========== =================================================================== =====================================================
부하(요청)	모든 서버가 부하를 균등하게 분배	                                반응성과 재사용성이 좋은 서버로 로드가 가중됨
연결비용	높음 (해당 서버의 순서가 되면 연결된 세션을 찾고 없으면 연결시도)   낮음 (재사용할 수 있는 세션이 없을 때만 연결)
재사용성	낮음 (서버 분배 우선)	                                            높음 (항상 연결된 세션을 우선 사용)
세션수	    많음 (각 서버마다 동시에 진행되는 HTTP 트랜잭션의 합)               적음 (동시에 진행되는 HTTP 트랜잭션 만큼 세션 존재)
=========== =================================================================== =====================================================


세션 재사용
====================================

원본서버가 Keep-Alive를 지원한다면 연결된 세션은 항상 재사용된다. 
하지만 세션을 재사용하여 보낸 요청에 대해 원본서버가 일방적으로 연결을 종료할 수 있다.
때문에 연결을 복구하느라 사용자 반응성이 늦어질 가능성이 있다.
특히 오랫동안 재사용하지 않은 세션의 경우 이러한 가능성은 더욱 높다. 
이를 방지하기 위하여 n초 동안 재사용되지 않은 세션에 대해서 자동으로 연결을 종료하도록 설정한다. ::

   <OriginOptions>
      <ReuseTimeout>60</ReuseTimeout>
   </OriginOptions>

-  ``<ReuseTimeout> (기본: 60초)`` 
   일정 시간동안 사용되지 않은 원본세션은 종료한다.
   0으로 설정하면 원본서버 세션을 재사용하지 않는다.
   
   

Range요청
====================================

한번에 다운로드 받는 컨텐츠 크기를 설정한다.
일반적으로 클라이언트가 요청한 콘텐츠에 대해서는 전체를 다운로드 받는다.
하지만 동영상처럼 클라이언트가 앞부분만을 주로 소비하는 경우에 다운로드 크기를 제한하면 불필요한 원본 트래픽를 줄일 수 있다. ::

   <OriginOptions>
      <PartSize>0</PartSize>
   </OriginOptions>

-  ``<PartSize> (기본: 0 MB)``
   0보다 크면 클라이언트가 요청한 지점부터 설정크기(MB) 만큼 Range요청으로 다운로드 한다.   

.. note:
   
   최초 콘텐츠를 다운로드할 때 Content-Length를 알 수 없으므로 Range요청을 할 수 없다.
   때문에 ``<PartSize>`` 가 설정되어 있다면 설정크기만큼만 다운로드 받고 연결을 종료한다.
      


전체 Range 초기화
====================================

일반적으로 원본서버로부터 처음 파일을 다운로드 할 때나 갱신확인 할 때는 다음과 같이 단순한 형태의 GET 요청을 보낸다. ::

    GET /file.dat HTTP/1.1
    
하지만 원본서버가 일반적인 GET요청에 대하여 항상 파일을 변조하도록 설정되어 있다면 원본파일 그대로를 Caching할 수 없어서 문제가 될 수 있다.

가장 대표적인 예는 Apache 웹서버가 mod_h.264_streaming같은 외부모듈과 같이 구동되는 경우이다.
Apache 웹서버는 GET요청에 대해서 항상 mod_h.264_streaming모듈을 통해서 응답한다.
클라이언트(이 경우에는 STON)는 원본파일 그대로가 아닌 모듈에 의해 변조된 파일을 서비스 받는다.

   .. figure:: img/conf_origin_fullrangeinit1.png
      :align: center
      
      mod_h.264_streaming모듈은 항상 원본을 변조한다.

Range요청을 사용하면 모듈을 우회하여 원본을 다운로드할 수 있다. ::

   <OriginOptions>
      <FullRangeInit>OFF</FullRangeInit>
   </OriginOptions>

-  ``<FullRangeInit>``

   - ``OFF (기본)`` 일반적인 HTTP요청을 보낸다.
   
   - ``ON`` 0부터 시작하는 Range요청을 보낸다. 
     Apache의 경우 Range헤더가 명시되면 모듈을 우회한다. ::
      
        GET /file.dat HTTP/1.1
        Range: bytes=0-
    
     최초로 파일 Caching할 때는 컨텐츠의 Range를 알지 못하므로 Full-Range(=0부터 시작하는)를 요청한다. 
     원본서버가 Range요청에 대해 정상적으로 응답(206 OK)하는지 반드시 확인해야 한다.

콘텐츠를 갱신할 때는 다음과 같이 **If-Modified-Since** 헤더가 같이 명시된다.
원본서버가 올바르게 **304 Not Modified** 로 응답해야 한다. ::

   GET /file.dat HTTP/1.1
   Range: bytes=0-
   If-Modified-Since: Sat, 29 Oct 1994 19:43:31 GMT
    
.. note::

   ``<FullRangeInit>`` 가 정상동작함을 확인한 웹서버들 목록.
    
   - Microsoft-IIS/7.5
   - nginx/1.4.2
   - lighttpd/1.4.32
   - Apache/2.2.22
    

.. _origin-httprequest:
    
원본요청 헤더
====================================

Host 헤더
---------------------

원본서버로 보내는 HTTP요청의 Host헤더를 설정한다.
별도로 설정하지 않은 경우 가상호스트 이름이 명시된다. ::

   <OriginOptions>
      <Host />
   </OriginOptions>

-  ``<Host>``
   원본서버로 보낼 Host헤더를 설정한다.
   원본서버에서 80포트 이외의 포트로 서비스하고 있다면 반드시 포트 번호를 명시해야 한다. ::
   
      <Host>www.example2.com:8080</Host>


클라이언트가 보낸 Host헤더를 원본으로 보내고 싶은 경우 *로 설정한다.


User-Agent 헤더
---------------------

원본서버로 보내는 HTTP요청의 User-Agent헤더를 설정한다. ::

   <OriginOptions>
      <UserAgent>STON</UserAgent>
   </OriginOptions>

-  ``<UserAgent> (기본: STON)``
   원본서버로 보낼 User-Agent헤더를 설정한다.


XFF(X-Forwarded-For) 헤더
---------------------

클라이언트와 원본서버 사이에 STON이 위치하면 원본서버는 클라이언트 IP를 얻을 수 없다.
때문에 STON은 원본서버로 보내는 모든 HTTP요청에 X-Forwarded-For헤더를 명시한다. ::

   <OriginOptions>
      <XFFClientIPOnly>OFF</XFFClientIPOnly>
   </OriginOptions>

-  ``<XFFClientIPOnly>``
   
   - ``OFF (기본)`` 클라이언트(IP: 128.134.9.1)가 보낸 XFF헤더에 클라이언트 IP를 추가한다.
     클라이언트가 XFF헤더를 보내지 않았다면 클라이언트 IP만 명시된다. ::
      
        X-Forwarded-For: 220.61.7.150, 61.1.9.100, 128.134.9.1
   
   - ``ON`` XFF헤더의 첫번째 주소만을 원본서버로 전송한다. ::
   
        X-Forwarded-For: 220.61.7.150


ETag 헤더 인식
---------------------

원본서버에서 응답하는 ETag인식여부를 설정한다. ::

   <OriginOptions>
      <OriginalETag>OFF</OriginalETag>
   </OriginOptions>

-  ``<OriginalETag>``
   
   - ``OFF (기본)`` ETag헤더를 무시한다.
   
   - ``ON`` ETag를 인식하며 컨텐츠 갱신시 If-None-Match헤더를 추가한다.

   
Redirect 추적
====================================

원본서버에서 Redirect계열(301, 302, 303, 307)로 응답하는 경우 Location헤더를 추적하여 콘텐츠를 요청한다. 

   .. figure:: img/conf_redirectiontrace.png
      :align: center
      
      클라이언트는 Redirect여부를 모른다.

::

   <OriginOptions>
      <RedirectionTrace>OFF</RedirectionTrace>
   </OriginOptions>

-  ``<RedirectionTrace>``

   - ``OFF (기본)`` 3xx 응답으로 저장된다.
   
   - ``ON`` Location헤더에 명시된 주소에서 콘텐츠를 다운로드 한다.
     형식에 맞지 않거나 Location헤더가 없는 경우에는 동작하지 않는다.
     무한히 Redirect되는 경우를 방지하기 위하여 1회만 추적한다.

