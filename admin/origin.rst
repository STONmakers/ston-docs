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



.. _origin_exclusion_and_recovery:

장애감지와 복구
====================================

Caching과정 중 원본서버에 장애가 발생하면 자동배제한다.
다시 안정화됐다고 판단하면 서비스에 투입한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <ConnectTimeout>3</ConnectTimeout>
   <ReceiveTimeout>10</ReceiveTimeout>
   <Exclusion>3</Exclusion>
   <Recovery Cycle="10" Uri="/" ResCode="0" Log="ON">5</Recovery>

-  ``<ConnectTimeout> (기본: 3초)``

   n초 이내에 원본서버와 접속이 이루어지지 않는 경우 접속실패로 간주한다.

-  ``<ReceiveTimeout> (기본: 10초)``

   정상적인 HTTP요청에도 불구하고 원본서버가 HTTP응답을 n초 동안 보내지 않는 경우 전송실패로 간주한다.

-  ``<Exclusion> (기본: 3회)``

   원본서버에서 연속적으로 n번 장애상황( ``<ConnectTimeout>`` 또는 ``<ReceiveTimeout>`` )이 발생하면 해당 서버를 유효 원본서버 목록에서 배제한다.
   배제 전 정상적인 통신이 이루어진다면 이 값은 다시 0으로 초기화된다.

-  ``<Recovery> (기본: 5회)``

   ``Cycle`` 마다 ``Uri`` 로 요청하여 원본서버가 ``ResCode`` 로 연속적으로 n회 응답하면 해당 서버를 복구한다.
   이 값을 0으로 설정하면 복구하지 않는다.

   -  ``Cycle (기본: 10초)`` 일정시간(초)마다 시도한다.

   -  ``Uri (기본: /)`` 요청을 보낼 Uri

   -  ``ResCode (기본: 0)`` 정상응답으로 처리할 응답코드.
      0인 경우 응답코드와 상관없이 응답이 오면 성공으로 간주한다.
      200으로 설정하면 응답코드가 반드시 200이어야 정상응답으로 처리한다.
      콤마(,)를 사용하여 유효한 응답코드를 멀티로 설정한다.
      200, 206, 404로 설정하면 응답코드가 이 중 하나인 경우 정상응답으로 처리한다.

   -  ``Log (기본: ON)`` 복구를 위해 사용된 HTTP Transaction을 :ref:`admin-log-origin` 에 기록한다.



.. _origin-health-checker:

Health-Checker
====================================

`장애감지와 복구`_ 는 Caching 과정 중 발생하는 장애에 대응한다.
``<Recovery>`` 는 응답코드를 받는 즉시 HTTP Transaction을 종료한다.
하지만 Health-Checker는 HTTP Transaction이 성공함을 확인한다. ::

   # vhosts.xml - <Vhosts><Vhost>

   <Origin>
      <Address> ... </Address>
      <HealthChecker ResCode="0" Timeout="10" Cycle="10"
                     Exclusion="3" Recovery="5" Log="ON">/</HealthChecker>
      <HealthChecker ResCode="200, 404" Timeout="3" Cycle="5"
                     Exclusion="5" Recovery="20" Log="ON">/alive.html</HealthChecker>
   </Origin>

-  ``<HealthChecker> (기본: /)``

   Health-Checker를 구성한다. 멀티로 구성이 가능하다.
   값으로 Uri를 지정하며, XML예외 문자의 경우 CDATA를 사용한다.

   -  ``ResCode (기본: 0)`` 올바른 응답코드 (콤마로 멀티 구성가능)

   -  ``Timeout (기본: 10초)`` 소켓연결부터 HTTP Transaction이 완료될 때까지 유효시간

   -  ``Cycle (기본: 10초)`` 실행주기

   -  ``Exclusion (기본: 3회)`` 연속 n회 실패 시 해당서버 배제

   -  ``Recovery (기본: 5회)`` 연속 n회 성공 시 해당서버 재투입

   -  ``Log (기본: ON)`` HTTP Transaction을 :ref:`admin-log-origin` 에 기록한다.

Health-Checker는 멀티로 구성할 수 있으며 클라이언트 요청과 상관없이 독립적으로 수행된다.
`장애감지와 복구`_ 나 다른 Health-Checker와도 정보를 공유하지 않고
자신만의 기준으로 배제와 투입을 결정한다.


.. _origin-use-policy:

원본주소 사용정책
====================================

원본주소(IP)는 다음 요소들에 의해 어떻게 사용될지 결정된다.

-  :ref:`env-vhost-activeorigin` 주소 형식(IP 또는 Domain)과 보조주소
-  `장애감지와 복구`_
-  `Health-Checker`_

서비스를 운영하다보면 원본주소가 배제/복구되는 일은 빈번하다.
STON은 IP테이블을 기반으로 원본주소를 사용하며 `origin-status`_ API를 통해 정보를 제공한다.

원본주소를 IP로 설정한 경우 매우 간단하다.

-  설정변경 이외에 IP목록을 변화시키는 요인은 없다.
-  TTL에 의해 IP주소가 만료되지 않는다.
-  장애/복구 모두 설정(IP주소)에 기반하여 동작한다.

원본주소를 Domain으로 설정하면 Resolving해서 IP를 얻어야 한다.
( :ref:`admin-log-dns` 에 기록된다.)
IP 목록은 동적으로 변경될 수 있으며 모든 IP는 TTL(Time To Live)동안만 유효하다.

-  Domain은 주기적으로(1~10초) Resolving한다.
-  Resolving결과를 통해 사용할 IP테이블을 구성한다.
-  모든 IP는 TTL만큼만 유효하며 TTL이 만료되면 사용하지 않는다.
-  같은 IP가 다시 Resolving되면 TTL을 갱신한다.
-  IP테이블은 비어서는 안된다. (TTL이 만료되었더라도) 마지막 IP들은 삭제되지 않는다.

원본주소를 Domain으로 설정하여도 장애/복구는 IP기반으로 동작한다.
여기서 미묘한 점이 있다.
DNS 클라이언트(=STON)는 Domain의 모든 IP 목록을 정확히 알 수 없다.
하지만 사용할 수 없는 IP들만으로 Domain을 구성할 경우 장애상태가 지속될 수 없다.

Domain주소 장애/복구 정책은 다음과 같다.

-  (Domain에 대해) 알고 있는 모든 IP주소가 배제(Inactive)되면 해당 Domain주소가 배제된다.
-  신규 IP가 Resolving되더라도 Domain이 배제되어 있다면 IP주소는 처음부터 배제된다.
-  모든 IP가 TTL 만료되더라도 배제된 Domain상태는 풀리지 않는다.
-  배제된 Domain에 속한 IP주소가 하나라도 복구되어야 해당 Domain은 다시 활성화된다.

다소 복잡한 내용이므로 `origin-status`_ API를 통해 서비스 동작상태에 대해 이해도를 높이는 것이 좋다.



.. _origin-status:

원본상태 모니터링
====================================

API를 통해 가상호스트의 원본상태를 모니터링한다. ::

   http://127.0.0.1:10040/monitoring/origin       // 모든 가상호스트
   http://127.0.0.1:10040/monitoring/origin?vhost=www.example.com

결과는 JSON형식으로 제공된다. ::

   {
       "origin" :
       [
           {
               "VirtualHost" : "example.com",
               "Address" :
               [
                   { "1.1.1.1" : "Active" },
                   { "1.1.1.2" : "Active" }
               ],
               "Address2" : [  ],
               "ActiveIP" :
               [
                   { "1.1.1.1" : 0 },
                   { "1.1.1.2" : 0 }
               ] ,
               "InactiveIP" : [ ]
           },
           {
               "VirtualHost" : "foobar.com",
               "Address" :
               [
                   { "origin.foobar.com" : "Active" }
               ],
               "Address2" : [  ],
               "ActiveIP" :
               [
                   { "5.5.5.5" : 21 },
                   { "5.5.5.6" : 60 },
                   { "5.5.5.7" : 37 }
               ],
               "InactiveIP" :
               [
                   { "5.5.5.8" : 10 },
                   { "5.5.5.9" : 184 }
               ]
           }
       ]
   }

-  ``VirtualHost`` 가상호스트 이름

-  ``Address`` :ref:`env-vhost-activeorigin` .
   설정주소가 사용중이라면 ``Active`` , (장애발생으로) 사용하고 있지 않다면 ``Inactive`` 로 표시된다.

-  ``Address2`` :ref:`env-vhost-standbyorigin` .
   설정주소를 사용중이라면 ``Active`` , 사용하고 있지 않다면 ``Inactive`` 로 표시된다.

-  ``ActiveIP`` 사용 중인 IP목록과 TTL.
   원본서버를 IP로 설정하면 ``Address`` 와 동일한 IP에 TTL은 0으로 표시된다.
   Domain으로 설정하면 Resolving결과에 따른다.
   다양한 IP와 TTL을 사용한다.

-  ``InactiveIP`` 사용하지 않는 IP목록과 TTL.
   사용하지 않더라도 복구 중이거나 HealthChecker에 의해 관리될 수 있다.
   해당 주소는 TTL 동안 복구되지 않으면 삭제된다.



.. _origin-status-reset:

원본상태 초기화
====================================

API를 통해 가상호스트의 원본서버 배제/복구를 초기화한다.
또한 현재 사용 중인 세션을 재사용하지 않고 새롭게 연결을 생성한다. ::

   http://127.0.0.1:10040/command/resetorigin       // 모든 가상호스트
   http://127.0.0.1:10040/command/resetorigin?vhost=www.example.com



.. _origin-busysessioncount:

과부하 판단
====================================

처음 요청되는 콘텐츠는 항상 원본서버에 요청해야 한다.
하지만 이미 Caching된 콘텐츠라면 좀 더 유연하게 대처할 수 있다.
원본서버가 과부하 상태라고 판단되면 갱신을 늦추어 원본부하를 높이지 않는다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <BusySessionCount>100</BusySessionCount>

-  ``<BusySessionCount> (기본: 100개)``
   원본서버와 HTTP트랜잭션을 진행 중인 세션 수가 일정개수를 넘으면 과부하 상태로 판단한다.
   과부하 상태에서 만료된 컨텐츠를 갱신하기 위해 원본서버로 접속하지 않도록 TTL을 :ref:`caching-policy-ttl` 중 ``<OriginBusy>`` 만큼 연장한다.
   무조건 원본서버로 요청이 가도록 하려면 이 값을 아주 크게 설정하면 된다.


.. _origin-balancemode:

원본 선택
====================================

원본서버 주소가 멀티(2개 이상)로 구성되어 있을 때 원본서버 선택정책을 설정한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <BalanceMode>RoundRobin</BalanceMode>

-  ``<BalanceMode> (기본: RoundRobin)``

   -  ``RoundRobin (기본)``
      모든 원본서버가 균등하게 요청을 받도록 Round-Robin으로 동작한다.
      연결된 Idle세션은 해당 서버로 요청이 필요할 때만 사용한다.

   -  ``Session``
      재사용할 수 있는 세션이 있다면 우선 사용한다.
      신규 세션이 필요하면 Round-Robin으로 할당한다.

   -  ``Hash``
      컨텐츠를 `Consistent Hashing <http://en.wikipedia.org/wiki/Consistent_hashing>`_ 알고리즘에 따라 원본서버로 분산하여 요청한다.
      서버가 선택되면 이미 연결된 세션을 재사용하며 없다면 신규로 접속한다.


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

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <ReuseTimeout>60</ReuseTimeout>

-  ``<ReuseTimeout> (기본: 60초)``
   일정 시간동안 사용되지 않은 원본세션은 종료한다.
   0으로 설정하면 원본서버 세션을 재사용하지 않는다.


.. _origin_partsize:

Range요청
====================================

한번에 다운로드 받는 컨텐츠 크기를 설정한다.
동영상처럼 앞 부분만이 주로 소비되는 컨텐츠의 경우 다운로드 크기를 제한하면 불필요한 원본 트래픽를 줄일 수 있다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <PartSize>0</PartSize>

-  ``<PartSize> (기본: 0 MB)``
   0보다 크면 클라이언트가 요청한 지점부터 설정크기(MB) 만큼 Range요청으로 다운로드 한다.


``<PartSize>`` 를 사용하는 또 다른 이유는 디스크 공간을 절약하기 위함이다.
기본설정으로 STON은 원본크기의 파일을 디스크에 생성한다.
하지만 ``<PartSize>`` 가 0이 아니라면 다운로드 되는만큼만 파일을 분할하여 저장한다.

예를 들어 1시간짜리 영상(600MB)을 1분(10MB)만 시청한 경우에 디스크 공간을 10MB만 사용한다.
공간을 절약하는 장점은 있지만 파일이 분할되어 저장되기 때문에 디스크 부하가 조금 높아진다.

.. note::

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

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <FullRangeInit>OFF</FullRangeInit>

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


.. _origin-wholeclientrequest:

클라이언트 요청 유지
====================================

원본에 요청할 때 클라이언트가 보낸 요청을 유지하도록 설정한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <WholeClientRequest>OFF</WholeClientRequest>

-  ``<WholeClientRequest>``

   - ``OFF (기본)`` Caching-Key를 원본에 요청할 URL로 사용한다.

   - ``ON`` 클라이언트가 요청한 URL로 원본에 요청한다.

Hit Ratio를 높이기 위해 다음 설정들을 통해 Caching-Key를 결정한다.

- :ref:`caching-policy-casesensitive`
- :ref:`caching-policy-applyquerystring`
- :ref:`caching-policy-post-method-caching`

이에 따라 원본서버로 요청하는 URL과 Caching-Key가 다음과 같이 결정된다.

============================================== ======================= ============================
설정                                           클라이언트 요청 URL       원본 요청URL / Caching-Key
============================================== ======================= ============================
:ref:`caching-policy-casesensitive` ``OFF``    /Image/LOGO.png         /image/logo.png
:ref:`caching-policy-casesensitive` ``ON``     /Image/LOGO.png         /Image/LOGO.png
:ref:`caching-policy-applyquerystring` ``OFF`` /view/list.php?type=A   /view/list.php
:ref:`caching-policy-applyquerystring` ``ON``  /view/list.php?type=A   /view/list.php?type=A
============================================== ======================= ============================

``<WholeClientRequest>`` 를 ``ON`` 으로 설정하면 다음과 같이 Caching-Key와 상관없이 클라이언트가 보낸 URL을 그대로 원본에 보낸다.

============================================== =================================== ============================
설정                                            클라이언트 / 원본 요청 URL           Caching-Key
============================================== =================================== ============================
:ref:`caching-policy-casesensitive` ``OFF``    /Image/LOGO.png                     /image/logo.png
:ref:`caching-policy-casesensitive` ``ON``     /Image/LOGO.png                     /Image/LOGO.png
:ref:`caching-policy-applyquerystring` ``OFF`` /view/list.php?type=A               /view/list.php
:ref:`caching-policy-applyquerystring` ``ON``  /view/list.php?type=A               /view/list.php?type=A
============================================== =================================== ============================

POST 요청을 캐싱하는 경우 원본서버로 요청할 때 클라이언트가 보낸 POST요청의 Body데이터가 수정없이 전송된다.

.. note::

   클라이언트가 보낸 URL을 그대로 보내기 때문에 :ref:`media-trimming` 처럼 부가기능을 위해 붙여진 QueryString도 그대로 원본서버로 전달된다.


.. _origin-httprequest:

원본요청 기본 Header
====================================

Host 헤더
---------------------

원본서버로 보내는 HTTP요청의 Host헤더를 설정한다.
별도로 설정하지 않은 경우 가상호스트 이름이 명시된다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <Host />

-  ``<Host>``
   원본서버로 보낼 Host헤더를 설정한다.
   원본서버에서 80포트 이외의 포트로 서비스하고 있다면 반드시 포트 번호를 명시해야 한다. ::

      # server.xml - <Server><VHostDefault><OriginOptions>
      # vhosts.xml - <Vhosts><Vhost><OriginOptions>

      <Host>www.example2.com:8080</Host>


클라이언트가 보낸 Host헤더를 원본으로 보내고 싶은 경우 *로 설정한다.


User-Agent 헤더
---------------------

원본서버로 보내는 HTTP요청의 User-Agent헤더를 설정한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <UserAgent>STON</UserAgent>

-  ``<UserAgent> (기본: STON)``
   원본서버로 보낼 User-Agent헤더를 설정한다.


클라이언트가 보낸 User-Agent헤더를 원본으로 보내고 싶은 경우 *로 설정한다.


XFF(X-Forwarded-For) 헤더
---------------------

클라이언트와 원본서버 사이에 STON이 위치하면 원본서버는 클라이언트 IP를 얻을 수 없다.
때문에 STON은 원본서버로 보내는 모든 HTTP요청에 X-Forwarded-For헤더를 명시한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <XFFClientIPOnly>OFF</XFFClientIPOnly>

-  ``<XFFClientIPOnly>``

   - ``OFF (기본)`` 클라이언트(IP: 128.134.9.1)가 보낸 XFF헤더에 클라이언트 IP를 추가한다.
     클라이언트가 XFF헤더를 보내지 않았다면 클라이언트 IP만 명시된다. ::

        X-Forwarded-For: 220.61.7.150, 61.1.9.100, 128.134.9.1

   - ``ON`` XFF헤더의 첫번째 주소만을 원본서버로 전송한다. ::

        X-Forwarded-For: 220.61.7.150


ETag 헤더 인식
---------------------

원본서버에서 응답하는 ETag인식여부를 설정한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <OriginalETag>OFF</OriginalETag>

-  ``<OriginalETag>``

   - ``OFF (기본)`` ETag헤더를 무시한다.

   - ``ON`` ETag를 인식하며 컨텐츠 갱신시 If-None-Match헤더를 추가한다.



.. _origin_url_rewrite:

원본요청 URL변경
====================================

캐싱을 목적으로 원본으로 보내는 HTTP요청의 URL을 변경한다. ::

   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <URLRewrite>
      <Pattern>/download/(.*)</Pattern>
      <Replace>/#1</Replace>
   </URLRewrite>
   // Pattern : /download/1.3.4
   // Replace : /1.3.4

   <URLRewrite>
      <Pattern>/img/(.*\.(jpg|png).*)</Pattern>
      <Replace>/#1/STON/composite/watermark1</Replace>
   </URLRewrite>
   // Pattern : /img/image.jpg?date=20140326
   // Replace : /image.jpg?date=20140326/STON/composite/watermark1

:ref:`handling_http_requests_url_rewrite` 와 같은 표현을 사용하지만
가상호스트마다 독립적으로 설정하기 때문에 가상호스트명을 입력하지 않는다.

.. note::

   바이패스되는 HTTP요청의 URL은 변경할 수 없다.
   ``<WholeClientRequest>`` 보다 우선한다.



.. _origin_modify_client:

원본요청 헤더변경
====================================

원본으로 HTTP요청을 보낼 때 조건에 따라 HTTP 헤더를 변경한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <ModifyHeader FirstOnly="OFF">OFF</ModifyHeader>

-  ``<ModifyHeader>``

   -  ``OFF (기본)`` 변경하지 않는다.

   -  ``ON`` 헤더 변경조건에 따라 헤더를 변경한다.

헤더 변경시점은 HTTP 요청패킷이 완성되어 원본서버로 전송하기 직전에 수행된다.
단, Range헤더는 변조할 수 없다.

이 기능은 :ref:`handling_http_requests_modify_client` 의 하위 기능이다.
헤더변경에는 $ORGREQ 키워드를 사용한다. ::

   # /svc/www.example.com/headers.txt

   $URL[/*.mp4], $ORGREQ[x-media-type: video/mp4], set
   $IP[1.1.1.1], $ORGREQ[user-agent: media_probe], put
   *, $ORGREQ[If-Modified-Since], unset
   *, $ORGREQ[If-None-Match], unset


.. note::

   If-Modified-Since 헤더와 If-None-Match 헤더를 ``unset`` 하면 TTL이 만료된 컨텐츠는 항상 다시 다운로드 한다.
