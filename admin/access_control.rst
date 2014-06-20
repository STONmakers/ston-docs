.. _access_control:

접근제어
******************

ACL(Access Control List)을 통해 서비스에 대한 클라이언트 접근을 제어할 수 있다.
접근제어는 접속단계에서 수행하는 서버 접근제어와 가상호스트마다 설정하는 가상호스트 
접근제어로 나뉜다.

.. toctree::
   :maxdepth: 2

서버 접근제어
====================================

클라이언트가 서버에 접속하면 IP정보를 통해 접근여부를 결정한다.
접속단계에서 처리되기 때문에 가장 확실하며 빠르다.
전역설정(server.xml)에 설정하며 모든 서비스에 가장 우선되어 적용된다. ::

    <Host>        
        <ServiceAccess Default="Allow">
            <Deny>192.168.7.9-255</Deny>
            <Deny>192.168.8.10/255.255.255.0</Deny>
        </ServiceAccess>
    </Host>

-  ``<ServiceAccess>``
   
   IP기반의 ACL을 설정한다. 
   IP, IP Range, Bitmask, Subnet 이상 네 가지 형식을 지원한다.
   설정파일은 순서를 인식하며 상위에 설정된 표현이 우선한다. 
   ``Default (기본: Allow)`` 속성은 일치하는 조건이 없을 때 허가여부이다.
   이 속성을 ``Deny`` 로 설정하면 하위에 ``<Allow>`` 로 허가할 IP주소를 명시해주어야 한다.
   
차단된 IP는 Deny로그에 기록된다.


GeoIP
---------------------

GeoIP를 사용하여 국가별로 접근을 차단할 수 있다. 
`GeoIP Databases <http://dev.maxmind.com/geoip/legacy/downloadable/>`_ 중 
Binary Databases를 `GEOIP_MEMORY_CACHE and GEOIP_CHECK_CACHE <http://dev.maxmind.com/geoip/legacy/benchmarks/>`_ 로 
링크하여 실시간으로 변경내용을 반영한다. ::

    <ServiceAccess GeoIP="/var/ston/geoip/">
        <Deny>AP</Deny>
        <Deny>GIN</Deny>
    </ServiceAccess>
    
``<ServiceAccess>`` 의 ``GeoIP`` 속성에 GeoIP Databases 경로를 설정한다.
국가코드는 `ISO 3166-1 alpha-2 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2>`_ 와 
`ISO 3166-1 alpha-3 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3>`_ 를 지원한다.

.. note::

    GeoIP는 파일명이 예약되어 있으므로 반드시 저장된 로컬경로를 입력하도록 설정한다. 
    또한 자동으로 변경이 반영되기 별도로 STON을 Reload하지 않아도 된다.

