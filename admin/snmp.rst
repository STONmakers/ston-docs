.. _snmp:

SNMP
******************

STON은 자체 SNMP(Simple Network Monitoring Protocol)를 통해 통계와 시스템 상태정보를 제공한다. 
가상호스트별로 실시간 통계와 최대 60분까지 "분" 단위의 평균 통계를 제공한다. 

- 별도의 패키지가 필요치 않다.
- snmpd를 별도로 실행할 필요가 없다.
- SNMP v1과 v2c를 지원한다.
- Community를 등록하는 절차는 없다. Public을 사용한다.

.. toctree::
   :maxdepth: 2



.. _snmp-var:

SNMP 변수
====================================

설정이나 사용자의 의도에 의하여 변경될 수 있는 값을 [변수명]으로 명시한다. 
예를 들어 디스크는 여러개가 존재할 수 있다. 
이 경우 각 디스크를 가리키는 고유 번호가 필요하며 입력된 순서대로 1부터 할당된다. 
이런 변수를 [diskIndex]로 명시한다. 

-  [diskIndex]

   Storage에 설정된 디스크를 의미한다. ::
   
      <Storage>
         <Disk>/cache1</Disk>
         <Disk>/cache2</Disk>
         <Disk>/cache3</Disk>
      </Storage>
      
   위와 같이 3개의 디스크가 설정된 환경에서 /cache1의 
   [diskIndex]는 1, /cache3의 [diskIndex]는 3을 가진다. 
   예를 들어 /cache1의 전체용량에 해당하는 OID는 
   system.diskInfo.diskInfoTotalSize.1 
   (1.3.6.1.4.1.40001.1.2.18.1.3.1이 된다. 
   마지막 .1은 첫번째 디스크를 의미한다.
   
-  [vhostIndex]

   가상호스트가 로딩될 때 자동으로 부여된다. ::
   
      <Vhosts>
         <Vhost Status="Active" Name="kim.com"> ... </Vhost>
         <Vhost Status="Active" Name="lee.com"> ... </Vhost>
         <Vhost Status="Active" Name="park.com" StaticIndex="10300"> ... </Vhost>
      </Vhosts>
   
   최초 위와 같이 3개의 가상호스트가 로딩되면 1부터 순차적으로 [vhostIndex]가 부여된다. 
   이후 가상호스트는 [vhostIndex]를 기억하며, 가상호스트가 삭제되더라도 [vhostIndex]는 변하지 않는다. 
   가상호스트의 삭제와 추가가 동시에 발생할 경우 삭제가 먼저 동작하며, 
   신규 추가된 가상호스트는 비어있는 [vhostIndex]를 부여 받는다.
   
   .. figure:: img/snmp_vhostindex.png
      :align: center
      
      [vhostIndex]의 동작방식

-  [diskMin], [vhostMin]

   시간(분)을 의미한다. 
   5는 5분의 평균을 의미하며 60은 60분의 평균을 의미한다. 
   이 값은 1(분)부터 60(분)까지 범위를 가지며 0은 실시간(1초) 데이터를 의미한다.
   
SNMP에서는 동적으로 값이 바뀔 수 있는 항목에 대하여 Table구조를 사용한다. 
예를 들어 "디스크 전체크기"는 디스크의 개수에 따라 제공하는 데이터 개수가 
달라지기 때문에 Table구조를 사용하여 표현해야 한다. 
STON은 모든 가상호스트에 대하여 "분"단위 통계를 제공한다. 
그러므로 [vhostMin].[vhostIndex]라는 다소 난해한 표현을 제공한다. 

이 표현은 가상호스트별로 원하는 "분" 단위의 통계를 볼 수 있다는 장점을 가지고 있지만 
변수가 2개이므로 Table구조로 표현하기 어렵다는 단점이 있다. 
이런 문제를 극복하기 위하여 [vhostMin]의 기본값을 설정하여 
SNMPWalk가 동작할 수 있도록 한다.


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
    
    

가상호스트/View 변수
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



기타 변수
---------------------

기타 변수를 설정한다. ::

    <SNMP GlobalMin="5" DiskMin="5" ConfCount="10" />
    
-  ``GlobalMin (기본: 5분, 최대: 60분)`` [globalMin] 값을 설정한다.

-  ``DiskMin (기본: 5분, 최대: 60분)`` [diskMin] 값을 설정한다.

-  ``ConfCount (기본: 10)`` 설정목록을 n개까지 열람한다. 
   1~100사이에서 지정 가능하다. 
   1은 현재 반영된 설정을 의미하며 2는 이전 설정을 의미한다. 
   100은 현재를 기준으로 99번 이전의 설정을 의미한다.
   


Community
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



