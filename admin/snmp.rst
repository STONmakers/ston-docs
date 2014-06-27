.. _snmp:

SNMP
******************

STON은 자체 SNMP(Simple Network Monitoring Protocol)를 통해 통계와 시스템 상태정보를 제공한다. 
가상호스트별로 실시간 통계와 최대 60분까지 "분" 단위의 평균 통계를 제공한다. 

.. toctree::
   :maxdepth: 2



.. _snmp-conf:

SNMP 설정
====================================

전역설정(serve



.. _snmp-conf:

SNMP 설정
====================================

전역설정(server.xml)을 통해 SNMP동작방식과 ACL을 설정한다. ::

    <Server>
        <Host>  
            <SNMP Port="161" Status="Inactive">
                <Allow>192.168.5.1</Allow>
                <Allow>192.168.6.0/24</Allow>    
            </SNMP>  
        </Host>
    </Server>

-  ``<SNMP>`` 속성을 통해 SNMPdml 동작방식을 설정한다.

   - ``Port (기본: 161)`` SNMP 서비스 포트
   
   - ``Status (기본: Inactive)`` SNMP를 활성화 하려면 이 값을 ``Active`` 로 설정한다.
   
-  ``<Allow>`` SNMP접근을 허가할 IP주소를 설정한다. 
    IP지정, IP범위지정, 비트마스크, 서브넷 이상 네 가지 형식을 지원한다. 
    접속한 소켓이 허가된 IP가 아니면 응답을 주지 않는다.
    

가상호스트/View 정보
---------------------

SNMP를 통해 제공되는 가상호스트/View 개수와 기본시간(분)을 설정한다. ::

    <SNMP VHostCount=0, VHostMin=5 ViewCount=0, ViewMin=5 />

-  ``VHostCount (기본: 0)`` 0일 경우 존재하는 가상호스트까지만 응답을 한다. 
   0보다 큰 값일 경우 가상호스트 존재 유무에 상관없이 설정된 가상호스트까지 응답한다. 
   
-  ``ViewCount (기본: 0)`` View에 적용. ( ``VHostCount`` 와 동일)
   
-  ``VHostMin (기본: 5분, 최대: 60분)`` [vhostMin] 값을 설정한다. 
   0~60까지의 값을 가진다. 
   0일 경우 실시간 데이터를 제공하하며 1~60사이인 경우 해당 분만큼의 평균값을 제공한다.
   
-  ``ViewMin (기본: 0)`` View에 적용. ( ``VHostMin`` 와 동일)

예를 들어 3개의 가상호스트가 설정되어 있는 환경에서 SNMPWalk의 동작방식이 달라진다.

- VHostCount=0인 경우 ::

    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.1 = STRING: "web.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.2 = STRING: "img.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.3 = STRING: "vod.winesoft.co.kr"
    
- VHostCount=5 경우 ::

    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.1 = STRING: "web.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.2 = STRING: "img.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.3 = STRING: "vod.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.4 = ""
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.5 = ""



Community 설정
---------------------

Community를 설정하여 허가된 OID에만 접근/차단되도록 설정한다. ::

    <SNMP>
        <Community Name="example1" OID="Allow">
            <OID>1.3.6.1.4.1.40001.1.4.1</OID>
            <OID>1.3.6.1.4.1.40001.1.4.2</OID>
            <OID>1.3.6.1.4.1.40001.1.4.4</OID>
        </Community>
        <Community Name="example2" OID="Deny">
            <OID>1.3.6.1.4.1.40001.1.4.3.1.11.11.10.1-61</OID>
        </Community>
    </SNMP>
    
-  ``<Community>`` Community를 설정한다.

   - ``Name`` Community 이름.
   
   - ``OID (기본: Allow)`` 하위 ``<OID>`` 태그의 값을 설정한다.
     속성 값이 ``Allow`` 라면 하위 ``<OID>`` 목록만 접근 가능하다. 
     반대로 속성 값이 ``Deny`` 라면 하위 <OID>목록에는 접근이 불가능하다.

명시적인 OID(1.3.6.1.4.1.40001.1.4.4)와 범위OID(1.3.6.1.4.1.40001.1.4.3.1.11.11.10.1-61) 표현이 가능하다. 
OID를 허용/차단할 경우 하위 모든 OID에 대해 같은 규칙이 적용된다.



기타
---------------------

::

    <SNMP GlobalMin="5" DiskMin="5" ConfCount="10" />
    
-  ``GlobalMin (기본: 5분, 최대: 60분)`` [globalMin] 값을 설정한다.

-  ``DiskMin (기본: 5분, 최대: 60분)`` [diskMin] 값을 설정한다.

-  ``ConfCount (기본: 10)`` 설정목록을 n개까지 열람한다. 
   1~100사이에서 지정 가능하다. 
   1은 현재 반영된 설정을 의미하며 2는 이전 설정을 의미한다. 
   100은 현재를 기준으로 99번 이전의 설정을 의미한다.