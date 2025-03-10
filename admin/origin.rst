.. _origin:

7장. 원본서버
******************

이 장에서는 STON과 원본서버의 관계에 대해 설명한다.
원본서버란 일반적으로 HTTP 규격을 준수하는 웹서버를 의미한다.
관리자라면 원본을 보호하기 위해 이번 장의 모든 내용을 숙지할 필요가 있다.
이를 바탕으로 원본장애에도 내구성을 갖춘 유연한 서비스를 구축할 수 있다.


.. note::

   - `[Q&A] STON Edge Server에서 말하는 ‘원본서버’는 무엇을 의미하나요? <https://www.youtube.com/watch?v=S2pxrv9gUy8&index=8&list=PLqvIfHb2IlKc0M8JZNIjus9BseHXuu3w->`_
   - `[Q&A] STON Edge Server를 설치한 후에 내 웹서버(원본서버)와는 어떻게 연결하나요? <https://www.youtube.com/watch?v=RJcYwqAqOrY&list=PLqvIfHb2IlKc0M8JZNIjus9BseHXuu3w->`_
   - `[Q&A] 원본서버에 장애가 발생하면 STON Edge Server는 어떻게 대처하나요? <https://www.youtube.com/watch?v=TfhdKB_ncTc&index=2&list=PLqvIfHb2IlKc0M8JZNIjus9BseHXuu3w->`_
   - `[Q&A] 배제된 원본서버는 어떻게 서비스에 재투입되나요? <https://www.youtube.com/watch?v=iDekbXavdxQ&index=4&list=PLqvIfHb2IlKc0M8JZNIjus9BseHXuu3w->`_
   - `[Q&A] STON Edge Server의 원본 부하분산은 어떻게 동작하나요? <https://www.youtube.com/watch?v=RqzH92YC9us&list=PLqvIfHb2IlKc0M8JZNIjus9BseHXuu3w-&index=3>`_


원본서버는 보호되어야 한다.
장애의 종류가 다양한 만큼 대처방안도 다양하다.
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
   <Exclusion All="ON">3</Exclusion>
   <Recovery Cycle="10" Uri="/" ResCode="0" Log="ON">5</Recovery>

-  ``<ConnectTimeout> (기본: 3초)``

   n초 이내에 원본서버와 접속이 이루어지지 않는 경우 접속실패로 간주한다.

-  ``<ReceiveTimeout> (기본: 10초)``

   정상적인 HTTP요청에도 불구하고 원본서버가 HTTP응답을 n초 동안 보내지 않는 경우 전송실패로 간주한다.

-  ``<Exclusion> (기본: 3회)``

   원본서버에서 연속적으로 n번 장애상황( ``<ConnectTimeout>`` 또는 ``<ReceiveTimeout>`` )이 발생하면 해당 서버를 유효 원본서버 목록에서 배제한다.
   배제 전 정상적인 통신이 이루어진다면 이 값은 다시 0으로 초기화된다.

   .. note::

      ``CDN v2.6.13`` , ``Enterprise v19.07.0`` 부터는 ``<Exclusion>`` 설정이 0이라면 원본서버를 배제하지 않는다.

   ``All`` 속성이 ``ON`` 이라면 장애가 발생한 모든 IP를 배제하지만, ``OFF`` 라면 마지막 IP는 배제하지 않는다.


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


.. _origin-health-checker-validation:

HTTP Validation 지원
--------------------------------

원본서버의 최초 ``200 OK`` 상태를 기억한 뒤 이후 HTTP Validation ( ``304 Not Modified`` )에 기반하여 ``Healthy`` 여부를 판단한다. ::

   # vhosts.xml - <Vhosts><Vhost><Origin>

   <HealthChecker ..(생략).. Validate="OFF">/</HealthChecker>


-  ``Validate (기본: OFF)``

   설정이 ``ON`` 일 경우 HTTP Validation 메커니즘( ``If-Modified-Since`` , ``If-None-Match`` ) 헤더를 추가한다. 
   반드시 유효 응답코드에 304를 추가해야 한다. ::

      <HealthChecker ResCode="200, 304" Validate="ON>/</HealthChecker>


.. _origin-health-checker-header:

응답헤더 추적
--------------------------------

원본서버가 응답한 특정 헤더를 추적하며 ``Healthy`` 여부를 판단한다. ::

   # vhosts.xml - <Vhosts><Vhost><Origin>

   <HealthChecker ..(생략)..  HealthyHeader="..." HealthyWhen="changed">/</HealthChecker>

-  ``HealthyHeader``
   추적할 원본응답 헤더

-  ``HealthyWhen (기본: changed)``
   응답헤더의 상태판단
   
   -  ``changed (기본)`` 이전 값에서 변경되었을 때만 ``Healthy`` 로 판단한다.
   
   -  ``unchanged`` 이전 값과 동일할때만 ``Healthy`` 로 판단한다.



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
-  Resolving을 통해 사용할 IP테이블을 구성한다.
-  모든 IP는 TTL만큼만 유효하며 TTL이 만료되면 사용하지 않는다.
-  같은 IP가 다시 Resolving되면 TTL을 갱신한다.
-  IP테이블은 비어서는 안된다. (TTL이 만료되었더라도) 마지막 IP들은 삭제되지 않는다.

IP의 TTL이 너무 길 경우 지나치게 많은 IP를 사용하게 되어 의도치 않은 결과를 만들 수 있다. 
이를 방지하기 위해 IP의 최대 TTL을 제한할 수 있다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <DnsMaxTTL Min="0">60</DnsMaxTTL>

-  ``<DnsMaxTTL> (기본: 60초)`` Resolving된 IP의 최대 사용시간(초)을 설정한다.
   이 값이 0일 경우 DNS로부터 제공받은 TTL을 그대로 사용한다.


간혹 너무 TTL이 짧은 경우 ``Min (기본: 0)`` 속성을 통해 최소 값을 보정할 수 있다.
예를 들어 ``Min="30"`` 이라면 TTL이 30미만인 경우 30으로 보정된다.



.. note::

   원본주소를 Domain으로 설정하여도 장애/복구는 IP기반으로 동작한다.
   Domain주소 장애/복구 정책은 다음과 같다.

   -  (Domain에 대해) 알고 있는 모든 IP주소가 배제(Inactive)되면 해당 Domain주소가 배제된다.
   -  신규 IP가 Resolving되더라도 Domain이 배제되어 있다면 IP주소는 처음부터 배제된다.
   -  모든 IP가 TTL 만료되더라도 배제된 Domain상태는 풀리지 않는다.
   -  배제된 Domain에 속한 IP주소가 하나라도 복구되어야 해당 Domain은 다시 활성화된다.

   다소 복잡한 내용이므로 `원본상태 모니터링`_ API를 통해 서비스 동작상태에 대해 이해도를 높이는 것이 좋다.


.. _origin-cache-control:

원본 Cache-Control 동작정책
====================================

원본서버가 다음과 같이 혼란스럽게 응답한 경우 STON의 동작에 대해 기술한다. ::

   Cache-Control: private, no-transform, max-age=900

   
``CDN - v2.7.11`` , ``Enterprise - v20.08.0`` 부터는 아래와 같이 동작한다.

   - ``private`` 은 ``cache`` 인가 ``no-cache`` 인가에 대해서 해석의 여지가 있다.
   - `Mozilla - Cache-Control <https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Cache-Control>`_ 에서는 다음과 같이 설명하고 있다. ::
   
      private
      
         응답이 단일 사용자를 위한 것이며 공유 캐시에 의해 저장되지 않아야 한다는 것을 나타냅니다. 
         사설 캐시는 응답을 저장할 수도 있습니다.

   - STON은 공유캐시 이기 때문에 ``private`` 이 있는 경우 ``no-cache`` 로 판단하고 ``no-cache`` TTL을 따르도록 한다.
   - 사용자에게 응답하는 경우에는 원본에서 준 ``max-age`` 를 그대로 줄 수 있도록 한다.

즉, ``no-cache`` 라고 하더라도 ``max-age`` 를 보정하지 않는 것이 올바른 정책이다.
필요하면 TTL 우선순위를 조정해서 처리 하는 것이 맞다.

참고로 이전 버전의 동작은 다음과 같다.

   - ``private`` 키워드가 있기 때문에 ``no-cache`` 로 인식하고 ``max-age`` 를 0으로 변경 한다.



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

   <BusySessionCount>0</BusySessionCount>

-  ``<BusySessionCount> (기본: 0)``
   원본서버와 HTTP트랜잭션을 진행 중인 세션 수가 일정개수를 넘으면 과부하 상태로 판단한다.
   과부하 상태에서 만료된 컨텐츠를 갱신하기 위해 원본서버로 접속하지 않도록 TTL을 :ref:`caching-policy-ttl` 중 ``<OriginBusy>`` 만큼 연장한다.
   무조건 원본서버로 요청이 가도록 하려면 이 값을 ``0`` 또는 아주 크게 설정하면 된다.


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


.. _origin-balancemode-url-suffix-ignore:

``Hash`` 분산 - Suffix 무시
-----------------------------------

STON의 많은 기능은 기존 URL 뒤에 명령어를 붙이는 형식이다. ::

   http://example.com/origin.jpg/...{명령어}...
   http://example.com/origin.jpg/dims/resize/100x100


자칫 이런 명령어들이 ``Hash`` 모드 원본분산 시 HIT율 저하의 원인이 되기도 한다.
예를 들어 다음 URL들은 모두 같은 원본파일을 변형하지만 URL이 다르다. ::

   http://example.com/origin.jpg/dims/resize/100x100
   http://example.com/origin.jpg/dims/strip/on/autorotate/on
   http://example.com/origin.jpg/dims/grayscale/true

결과적으로 각기 다른 원본서버가 선택되면 불필요한 원본부하가 발생한다.


이런 경우를 방지하기 위해 URL 중 매칭되는 Suffix를 무시하도록 설정한다. ::


   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <BalanceHashRules>
      <IgnoreSuffix>/dims/</IgnoreSuffix>
      <IgnoreSuffix>?start=</IgnoreSuffix>
   </BalanceHashRules>


-  ``<BalanceHashRules>`` 원본서버 ``Hash`` 분산시에만 동작하며 URL을 Hash할 때 ``<IgnoreSuffix>`` 와 매칭되는 영역 이후를 모두 무시한다.

위와 같이 ``<IgnoreSuffix>/dims/</IgnoreSuffix>`` 를 설정해 두면 예제의 3 URL들은 모두 같은 원본서버를 선택한다. ::

   http://example.com/origin.jpg    # /dims/resize/100x100 무시
   http://example.com/origin.jpg    # /dims/strip/on/autorotate/on 무시
   http://example.com/origin.jpg    # /dims/grayscale/true 무시


``<BalanceHashRules>`` 은 원본서버 선택에만 관여할 뿐 URL을 변형시키는 것은 아니다.




.. _origin-balancemode-on-counter:

카운터 기반 분산
-----------------------------------

요청수에 기반하여 동적으로 원본 분산정책을 적용한다.

.. note::

   이 기능 동작을 위해서는 :ref:`monitoring_counter` 가 활성화되어 있어야 한다. 


::


   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <BalanceMode>Hash
     <OnCounter AccessGt="40">
       <Pattern><![CDATA[/live/*.ts]]></Pattern>
       <Pattern><![CDATA][/live/*.m4v]]></Pattern>
       <Pattern><![CDATA][/live/*.m4a]]></Pattern>
       <Action>RoundRobin</Action>
     </OnCounter>
     <OnCounter AccessGt="10">
       <Pattern><![CDATA[/live/*.mp4]]></Pattern>
       <Action>RoundRobin</Action>
     </OnCounter>
   </BalanceMode>
   

원본을 선택하는 시점에 요청수(=카운터의 값)가 ``<OnCounter>`` 의 ``AccessGt`` 보다 크다면 ``<Action>`` 의 정책을 사용한다.
``<OnCounter>`` 하위에 ``<Pattern>`` 리스트를 작성하여 카운터를 적용할 URL을 정의한다.



.. _origin-retry:

원본 재시도
====================================

캐싱요청이 실패했을 때(=트랜잭션 전개되지 않음) 시도하지 않은 다른 원본이 존재한다면 재시도한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <Retry>0</Retry>


-  ``Retry (기본: 0회)``

   -  ``0`` 재시도하지 않는다.
   -  ``1 이상`` 원본 실패 시 설정된 횟수만큼 다른 서버로 요청을 재시도한다.
   -  ``all`` 모든 서버 IP를 순차적으로 재시도한다.

.. warning::

   재시도하는 만큼 클라이언트 대기 시간이 지연되어 서비스 품질에 악영향을 줄 우려가 있음에 주의한다.


.. _origin-session-reuse:

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
동영상처럼 앞 부분만이 주로 소비되는 컨텐츠의 경우 다운로드 크기를 제한하면 불필요한 원본 트래픽을 줄일 수 있다. ::

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



.. _origin-fullrangeinit:

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

   -  ``HEAD`` :ref:`origin-fullrangeinit-head`

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


.. _origin-fullrangeinit-head:

``HEAD`` 메소드 초기화
------------------------

원본서버가 ``Range`` 가 아닌 ``GET`` 메소드를 처리하지 못하는 경우 ``HEAD`` 메소드를 이용하여 캐싱객체를 초기화할 수 있다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <FullRangeInit>HEAD</FullRangeInit>


위와 같이 설정하면 객체 캐싱과 갱신과정에 ``HEAD`` 메소드가 사용된다. ::


   # 최초 요청
   ... HEAD ... /sample.mp3 ... 200 ...
   ... GET ... /sample.mp3 ... 206 ...

   # Expire 후 요청
   ... HEAD ... /sample.mp3 ... 304 ...

   # Purge 후 요청
   ... HEAD ... /sample.mp3 ... 200 ...
   ... GET ... /sample.mp3 ... 206 ...


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



.. _origin_header_if_range:

If-Range 헤더
---------------------

원본에 Range요청을 보낼 때 If-Range헤더를 추가하여 요청한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <IfRange>OFF</IfRange>

-  ``<IfRange>``

   -  ``OFF (기본)`` Range요청 시 If-Range헤더를 추가하지 않는다.

   -  ``ON`` Range요청 시 If-Range헤더를 추가한다.

If-Range 헤더에 의해 원본서버가 ``206 Partial Content`` 가 아닌 ``200 OK`` 응답을 주는 경우 다음과 같이 전개된다.

-  클라이언트가 요청한 파일은 무효화 처리
-  해당 클라이언트 요청은 실패
-  동일한 파일이 요청될 때 신규 캐싱 서비스


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

:ref:`adv-vhost-url-rewrite` 와 같은 표현을 사용하지만
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
헤더변경에는 ``$ORGREQ`` 키워드를 사용한다. ::

   # /svc/www.example.com/headers.txt

   $URL[/*.mp4], $ORGREQ[x-media-type: video/mp4], set
   $IP[1.1.1.1], $ORGREQ[user-agent: media_probe], put
   *, $ORGREQ[If-Modified-Since], unset
   *, $ORGREQ[If-None-Match], unset

   # #PROTOCOL 키워드를 통해 클라이언트가 요청한 프로토콜을 헤더에 추가한다.
   $URL[*], $ORGREQ[X-Forwarded-Proto: #PROTOCOL], set

   # $REQ.header-name을 이용해 클라이언트 요청을 원본으로 보낼 수 있다.
   $URL[/account/], $ORGREQ[cookie: $REQ.Cookie], set

   # #HOSTNAME 키워드를 통해 요청을 보내는 호스트(서버) 이름을 헤더에 추가한다.
   $URL[*], $ORGREQ[X-Req-Hostname: #HOSTNAME], set


.. note::

   If-Modified-Since 헤더와 If-None-Match 헤더를 ``unset`` 하면 TTL이 만료된 컨텐츠는 항상 다시 다운로드 한다.



.. _origin_aws_s3_authentication:

AWS S3 인증
====================================

AWS S3 인증스펙인 `Authenticating Requests (AWS Signature Version 4) <https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/API/sig-v4-authenticating-requests.html>`_ 를 지원한다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <Authorization Type="AWS-S3" Status="Inactive">
       <AccessKeyID>KeyID</AccessKeyID>
       <SecretAccessKey>seceret-key</SecretAccessKey>
       <Region>seoul</Region>
   </Authorization>

``<Authorization>`` 의 ``Status`` 속성이 ``Active`` 인 경우에 동작한다. 
`Authenticating Requests (AWS Signature Version 4) <https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/API/sig-v4-authenticating-requests.html>`_ 동작에 필요한 세부 설정 ``<AccessKeyID>`` , ``<SecretAccessKey>`` , ``<Region>`` 을 구성한다.
만약 모든 가상호스트의 원본서버가 AWS S3에 존재한다면 기본 가상호스트 ``<VHostDefault>`` 로도 설정이 가능하다.


.. note::

   읽기 권한인 GET 메소드만을 지원한다. 쓰기(POST, PUT, FETCH)는 캐시서버의 목적에 부합하지 않는다.



.. _origin_dynamic:

동적 원본분기
====================================

가상호스트는 :ref:`env-vhost-activeorigin` 를 대신하기에 1:1의 관계가 기본이다.
가상호스트 안에는 원본팜이 존재하지만 1:1의 관계이기 때문에 특별한 이유가 없다면 표기하지 않는다.

   .. figure:: img/origin_dynamic01.png
      :align: center

      원본팜은 항상 존재한다.


동적 원본분기는 클라이언트 요청 패턴에 따라 동적으로 원본팜을 생성하는 기능으로 다음과 같은 상황에서 효과적이다.

.. note::

   -  라이브 방송처럼 원본서버가 특정 이벤트동안만 운영될 때
   -  원본서버 정보를 사전에 알 수 없을 때
   -  단일 서비스 도메인으로 URL path로 원본이 구분될 때
   -  수백 개가 넘는 가상호스트를 사전에 구성/분기 작업이 어려울 때


.. figure:: img/origin_dynamic02.png
   :align: center

   한 가상호스트 안에 독립된 원본팜을 멀티로 구성한다.

::

   # vhosts.xml - <Vhosts><Vhost><Origin>

   <Dynamic Status="Active" KeepAlive="60">
      <MatchingList>
         <Item><![CDATA[$URL[/ko-kr/live/*/*], #1]]></Item>
         <Item><![CDATA[$URL[/*?ch=*&*], #2]]></Item>
      </MatchingList>
      <Origin Protocol="HTTP">
         <Address>#ORGKEY.mylive.com</Address>
      </Origin>
   </Dynamic>


.. warning::
   
   설정위치가 :ref:`env-vhost-activeorigin` ``<Origin>`` 하위임을 주의한다. ::

      # vhosts.xml

      <Vhosts>
         <Vhost Name="www.example.com">
            <Origin>
               <Dynamic Status="Active" KeepAlive="60">
                  ...
               </Dynamic>
            </Origin>
         </Vhost>
      </Vhosts>
   

-  ``<Dynamic>``  

   -  ``Status (기본: Inactive)`` 값이 ``Active`` 인 경우 활성화된다. 서비스 중 변경이 가능하다.

   -  ``KeepAlive (기본: 1800초, 최대: 86400초)`` 동적으로 생성된 원본팜이 일정 시간동안 접근되지 않는다면 파괴한다.

   -  ``<MatchingList>`` 동적 원본을 생성할 패턴을 하위 ``<Item>`` 으로 구성한다. ``<Item>`` 을 통해 동적 원본에서 사용할 키를 추출한다. ``#1 ~ #9`` 까지 사용이 가능하다. ::
          
          # /ko-kr/live/ch01/0000-0000-0001/test 요청시
          패턴: /ko-kr/live/*/*
          키: ch01 (#1)

          # /9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d?ch=ston 요청시
          패턴: /*?ch=*&*
          키: ston (#2)
          
      
      일치하는 패턴이 없다면 기존대로 지정된 :ref:`env-vhost-activeorigin` 를 사용한다. 

   -  ``<Origin>`` 동적 원본의 설정으로 ``<MatchingList>`` 에서 매칭된 키를 ``#ORGKEY`` 키워드로 사용할 수 있다.  ::
          
          # /ko-kr/live/ch01/0000-0000-0001/test 요청시 원본 형상
          <Origin Protocol="HTTP">
            <Address>ch01.mylive.com</Address>
          </Origin>

          # /9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d?ch=ston 요청시
          <Origin Protocol="HTTP">
            <Address>ston.mylive.com</Address>
          </Origin>


각각의 원본팜은 원본 사용정책 ``<OriginOptions>`` 는 공유하지만, 객체 ``<Origin>`` 는 완전히 독립된다.
따라서 다음과 같이 독립적인 :ref:`origin-health-checker` 도 구성 가능하다. ::

   # vhosts.xml - <Vhosts><Vhost><Origin>

   <Dynamic Status="Active" KeepAlive="60">
      <MatchingList>
         <Item><![CDATA[$URL[/ko-kr/live/*/*], #1]]></Item>
         <Item><![CDATA[$URL[/*?ch=*&*], #2]]></Item>
      </MatchingList>
      <Origin Protocol="HTTP">
         <Address>#ORGKEY.mylive.com</Address>
         <Address2>#ORGKEY-backup.mylive.com</Address>
         <HealthChecker ResCode="0" Timeout="10" Cycle="10" Exclusion="3" Recovery="5" Log="ON">/</HealthChecker> 
         <HealthChecker ResCode="200, 404" Timeout="3" Cycle="5" Exclusion="5" Recovery="20" Log="ON">/alive.html</HealthChecker>
      </Origin>
   </Dynamic>


각 원본팜마다 IP 구성, :ref:`origin-session-reuse` , :ref:`origin_exclusion_and_recovery` , :ref:`origin-health-checker` 등이 독립적으로 동작한다.

.. note::

   각 원본팜의 생성/파괴는 :ref:`admin-log-info` 에 기록된다. ::

      2024-09-12 18:18:36 [foo.com] dynamic origin <ch01> created: <Origin Protocol="HTTP"><Address>cd01.mylive.com</Address><Address2>cd01-backup.mylive.com</Address><HealthChecker ResCode="0" Timeout="10" Cycle="10" Exclusion="3" Recovery="5" Log="ON">/</HealthChecker><HealthChecker ResCode="200, 404" Timeout="3" Cycle="5" Exclusion="5" Recovery="20" Log="ON">/alive.html</HealthChecker></Origin>
      2024-09-12 18:48:36 [foo.com] dynamic origin <ch01> destroyed


