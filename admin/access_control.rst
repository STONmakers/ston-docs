.. _access-control:

15장. 접근제어
******************

이 장에서는 원치않는 클라이언트 접근을 차단하는 방법에 대해 설명한다.
접근차단은 보통 ACL(Access Control List)에 차단목록(Black-list)을 작성하지만 설정편의상 허용목록(White-list)을 작성하기도 한다.

접근제어는 접속단계에서 수행하는 서버 접근제어와 가상호스트마다 설정하는 가상호스트 접근제어로 나뉜다.
수준별로 시점과 판단기준이 다르므로 효과적인 차단시점을 결정해야 한다.
접근제어의 동작은 모두 로그에 기록된다.

.. toctree::
   :maxdepth: 2


.. _access-control-serviceaccess:

서버 접근제어
====================================

클라이언트가 서버에 접속하는 순간 IP정보를 통해 차단여부를 결정한다.
접속단계에서 처리되기 때문에 가장 확실하며 빠르다.
전역설정(server.xml)에 설정하며 가장 높은 우선순위를 가진다. ::

   # server.xml - <Server><Host>

   <ServiceAccess Default="Allow">
      <Deny>192.168.7.9-255</Deny>
      <Deny>192.168.8.10/255.255.255.0</Deny>
   </ServiceAccess>

-  ``<ServiceAccess>``
   IP기반의 ACL을 설정한다.
   IP, IP Range, Bitmask, Subnet 이상 네 가지 형식을 지원한다.
   순서를 인식하며 상위에 설정된 표현이 우선한다.
   ``Default (기본: Allow)`` 속성은 일치하는 조건이 없을 때 처리방법이다.
   이 속성을 ``Deny`` 로 설정하면 하위에 ``<Allow>`` 로 허가할 조건들을 명시해주어야 한다.

차단된 IP는 :ref:`admin-log-deny` 에 기록된다.


.. _access-control-geoip:

GeoIP
====================================

GeoIP를 사용하여 국가별로 접근을 차단할 수 있다.
`GeoIP Databases <http://dev.maxmind.com/geoip/legacy/downloadable/>`_ 중
Binary Databases를 `GEOIP_MEMORY_CACHE and GEOIP_CHECK_CACHE <http://dev.maxmind.com/geoip/legacy/benchmarks/>`_ 로
링크하여 실시간으로 변경내용을 반영한다. ::

   # server.xml - <Server><Host>

   <ServiceAccess GeoIP="/var/ston/geoip/">
      <Deny>AP</Deny>
      <Deny>GIN</Deny>
   </ServiceAccess>

``<ServiceAccess>`` 의 ``GeoIP`` 속성에 GeoIP Databases 경로를 설정한다.
국가코드는 `ISO 3166-1 alpha-2 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2>`_ 와
`ISO 3166-1 alpha-3 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3>`_ 를 지원한다.

.. note::

   GeoIP는 파일명이 예약되어 있으므로 반드시 저장된 로컬경로를 입력하도록 설정한다.
   또한 자동으로 변경이 반영되기 때문에 별도로 설정을 Reload하지 않아도 된다.


GeoIP가 설정되어 있다면 해당 디렉토리에 저장된 파일목록을 조회한다.
설정되어 있지 않다면 404 NOT FOUND로 응답한다. ::

   http://127.0.0.1:10040/monitoring/geoiplist

결과는 JSON형식으로 제공된다. ::

   {
       "version": "2.0.0",
       "method": "geoiplist",
       "status": "OK",
       "result":
       {
           "path" : "/usr/ston/geoip/",
           "files" :
           [
               {
                   "file" : "GeoIP.dat",
                   "size" : 766255
               },
               {
                   "file" : "GeoLiteCity.dat",
                   "size" : 12826936
               }
           ]
       }
   }


.. _access-control-vhost:

가상호스트 접근제어
====================================

가상호스트별로 접근을 제어한다.
클라이언트가 HTTP요청을 보냈을 때 차단여부를 결정한다.
왜냐하면 HTTP요청을 보내지 않는다면 가상호스트를 찾을 수 없기 때문이다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <AccessControl Default="Allow" DenialCode="401">OFF</AccessControl>

-  ``<AccessControl>``

   - ``OFF (기본)`` ACL이 활성화되지 않는다. 모든 클라이언트 요청을 허가한다.

   - ``ON`` ACL이 활성화된다.
     차단된 요청에 대해서는 ``DenialCode`` 속성에 설정된 응답코드로 응답한다.
     ``Default (기본: Allow)`` 속성이 ``Allow`` 라면 ACL은 거부목록이 된다.
     반대로 ``Deny`` 라면 ACL은 허가목록이 된다.

Deny된 요청은 :ref:`admin-log-access` 에 TCP_DENY로 기록된다.


.. _access-control-vhost_acl:

가상호스트 ACL
---------------------

모든 클라이언트 HTTP요청에 대하여 허용/거부/Redirect 여부를 판단한다.
각 조건마다 별도로 응답코드를 설정할 수도 있다.
Redirect된 요청에 대해서는 **302 Moved temporarily** 로 응답한다.
ACL은 /svc/{가상호스트 이름}/acl.txt에 설정한다. ::

   # /svc/www.example.com/acl.txt
   # 구분자는 콤마(,)이며 {조건},{키워드 = allow | deny | redirect} 순서로 표기한다.
   # deny일 경우 키워드 뒤에 응답코드를 명시할 수 있다.
   # 명시하지 않으면 ``<AccessControl>`` 의 ``DenialCode`` 를 사용한다.
   # redirect일 경우 키워드 뒤에 이동시킬 URL을 명시한다. (Location헤더의 값으로 명시)
   # n 개의 조건을 결합(AND)하기 위해서는 &를 사용한다.

   $IP[192.168.1.1], allow
   $IP[192.168.2.1-255]
   $IP[192.168.3.0/24], deny
   $IP[192.168.4.0/255.255.255.0]
   $IP[AP] & !HEADER[referer], allow
   $IP[GIN], redirect, /page/illegal_access.html
   $HEADER[cookie: *ILLEGAL*], deny, 404
   $HEADER[via: Apache]
   $HEADER[x-custom-header]
   $HEADER[referer:], redirect, http://another-site.com
   !HEADER[referer] & !HEADER[user-agent] & !HEADER[host], deny
   $URL[/source/public.zip], allow
   $URL[/source/*]
   /profile.zip, deny, 500
   /secure/*.dat

조건은 IP, GeoIP, Header, URL 4가지로 설정이 가능하다.

-  **IP**
   $IP[...]로 표기하며 IP, IP Range, Bitmask, Subnet 네 가지 형식을 지원한다.

-  **GeoIP**
   $IP[...]로 표기하며 반드시 GeoIP설정이 되어 있어야 동작한다.

-  **Header**
   $HEADER[Key : Value]로 표기한다.
   Value는 명확한 표현과 패턴을 인식한다.
   $HEADER[Key:]처럼 구분자는 있지만 Value가 빈 문자열이라면 요청 헤더의 값이 비어 있는 경우를 의미한다.
   $HEADER[Key]처럼 구분자 없이 Key만 명시되어 있다면 Key에 해당하는 헤더의 존재유무를 조건으로 판단한다.

-  **URL**
   $URL[...]로 표기하며 생략이 가능합니다. 명확한 표현과 패턴을 인식합니다.

$는 "조건에 맞다면 ~ 한다"를 의미하지만 !는 "조건에 맞지 않는다면 ~ 한다"를 의미한다.
다음과 같이 부정조건으로 지원한다. ::

   # 국가가 KOR이 아니라면 deny한다.
   !IP[KOR], deny

   # referer헤더가 존재하지 않는다면 deny한다.
   !HEADER[referer], deny

   # /secure/ 경로 하위가 아니라면 allow한다.
   !URL[/secure/*], allow

Redirect 할 때 클라이언트가 요청한 URI가 필요할 수 있다.
이런 경우 ``#URI`` 키워드를 사용한다. ::

   # referer헤더가 존재하지 않는다면 example.com에 요청 URI를 붙여서 Redirect한다.
   # 클라이언트 요청은 /로 시작하기 때문에 #URI 앞에 /를 붙이지 않도록 주의한다.
   !HEADER[referer], redirect, http://example.com#URI
