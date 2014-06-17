.. _environment:

전역설정과 상속
******************

설정은 크게 전역(server.xml)과 가상호스트(vhosts.xml)으로 나뉜다.

   .. figure:: img/conf_files.png
      :align: center

      3개의 .xml파일이 전부입니다.

이 장에서는 전역설정에 대해 알아보며 가상호스트가 전역설정을 상속받는 방식에 대해 설명한다.

.. toctree::
   :maxdepth: 2

server.xml
====================================

실행파일과 같은 경로에 존재하는 server.xml 이 유일한 전역설정 파일이다.
XML형식으로 누구나 간단히 편집할 수 있다. ::

    <Server>
        <Host> ... </Host>
        <Cache> ... </Cache>
        <VHostDefault> ... </VHostDefault>
        <Https> ... </Https>
    </Server>
    
여기서는 전역설정의 모든 부분에 대해 설명하지는 않는다. 예를 들어 접근제어나 SNMP등은 
각 주제를 다루는 장에서 설명한다. 기본적인 구조와 기능에 대한 이해를 목적으로 한다.

<Host>
------------------------------------------------

관리목적의 기능을 설정한다. ::

    <Host>
        <Name>stream_07</Name>
        <Admin>admin@winesoft.co.kr</Admin>
        <Manager Port="10040" HttpMethod="ON" Role="Admin" UploadMultipartName="confile">
            <Allow>192.168.1.1</Allow>
            <Allow Role="Admin">192.168.2.1-255</Allow>
            <Allow Role="User">192.168.3.0/24</Allow>
            <Allow Role="Looker">192.168.4.0/255.255.255.0</Allow>
        </Manager>
    </Host>

-  ``Name``
    서버 이름을 설정합니다. 이름이 입력되지 않으면, 시스템에 설정된 시스템 이름이 사용됩니다.
-  ``Admin``
    관리자 정보(메일 또는 이름)를 설정합니다. 이 항목은 SNMP 조회목적으로만 사용됩니다.
-  ``Manager``
    관리용도로 사용할 매니저 포트와 ACL(Access Control List)을 설정한다. ACL은 IP, IP범위, 
    BitMask, Subnet 이상 네 가지 형식을 지원한다. 접속한 세션이 Allow로 접근이 허가된 
    IP가 아니면 강제로 접속을 종료한다. API를 호출하는 IP가 매니저 허가 목록에 반드시
    등록되어 있어야 한다.
    
    접근조건에 따라 접근권한(Role)을 설정할 수 있다. 접근권한이 없는 요청에 
    대해서는 401 Unauthorized로 응답한다. Allow조건에 Role을 명시적으로 선언하지 
    않았을 경우 Manager의 Role속성이 적용된다.
    
    - ``Admin`` 모든 API호출이 가능하다.
    - ``User`` Monitoring과 Graph계열 API만 호출할 수 있다.
    - ``Looker`` Graph계열 API만 호출할 수 있다.
    
    서비스포트(80)로 HTTP Method를 호출하는 경우 HttpMethod속성의 영향을 받는다. 
    이 설정이 ON인 경우 서비스포트로 요청되었더라도 HTTP Method라면 ACL의 영향을 받는다. 
    반대로 이 설정이 OFF인 경우 ACL의 영향을 받지 않는다. 
    
    설정파일을 POST(Multipart방식)로 업로드할 때 UploadMultipartName속성의 값을 사용한다.

<Cache>
------------------------------------------------

Cache서비스 모듈을 설정한다. ::

    <Cache>
        <Cleanup>
            <Time>02:00</Time>
            <Age>0</Age>
        </Cleanup>
        <Storage DiskFailSec="60" DiskFailCount="10" OnCrash="hang">
            <Disk>/user/cache1</Disk>    
            <Disk>/user/cache2</Disk>    
            <Disk Quota="100">/user/cache3</Disk>
        </Storage>        
        <HttpClientSession>
            <Init>20000</Init>
            <TopUp>6000</TopUp>
            <Max>60000</Max>
        </HttpClientSession>
        <Listen>0.0.0.0</Listen>        
        <ConfigHistory>30</ConfigHistory>
    </Cache>

-  ``Cleanup``
    하루에 한 번 시스템 최적화를 수행한다. 서비스 품질저하를 방지하기 위해 
    최적화는 조금씩 점진적으로 수행된다.

    - ``Time`` Cleanup이 수행될 시간을 설정한다. 오후 11시 10분을 설정하고 
    싶다면 23:10으로 설정한다.
    
    - ``Age (단위: 일)`` 0보다 큰 경우, 일정 기간동안 한번도 접근되지 않은 콘텐츠를 삭제한다.
    디스크를 미리 확보하여 서비스 시간 중 디스크 부족이 발생하지 않게 한다.
    
-  ``Storage``
    콘텐츠를 저장할 디스크를 설정한다. 디스크 개수에 제한은 없다. 각 디스크마다 
    최대 캐싱용량(Quota, 단위: GB)을 설정할 수 있다. 최대 캐싱용량을 설정하지 않아도 
    디스크가 꽉 차지 않도록 오래된 컨텐츠를 자동으로 삭제한다.
    
    디스크는 장애가 가장 많이 발생하는 장비이므로 장애조건을 설정해야 한다.
    DiskFailSec(초)동안 DiskFailCount만큼 디스크 작업이 실패하면 해당 디스크는 자동으로 
    배제된다. 배제된 디스크 상태는 "Invalid"로 제공된다. 모든 디스크가 배제될 수도 있는데
    이 때의 동작방식은 OnCrash속성을 통해 설정 가능하다.
    
    - ``hang`` 장애 디스크를 모두 재투입한다. 정상 서비스를 기대한다기 보다는 복구되기 전까지 트래픽을 버티려는 목적이 강하다.    
    - ``bypass`` 모든 요청을 원본서버으로 바이패스 한다. 디스크가 복구되면 즉시 STON이 서비스를 처리한다.
    - ``selfkill`` STON을 종료시킨다.
    
    
    
<VHostDefault> 설정
------------------------------------------------

모든 서비스는 가상호스트를 기반으로 완전히 분리되어 동작합니다. 
모든 가상호스트는 <VHostDefault>설정을 상속받습니다. 설정유형에 따라 
캐싱옵션(Options), 원본옵션(OriginOptions), 통계(Stats), 로그(Log)로 나뉩니다. ::

    <VHostDefault>
        <Options> ... </Options>  
        <OriginOptions> ... </OriginOptions>  
        <Media> ... </Media>  
        <Stats> ... </Stats>  
        <Log> ... </Log>
    </VHostDefault>


<Https> 설정
------------------------------------------------

HTTPS 서비스를 구성합니다. 자세한 내용은 **HTTPS 구성하기**를 참고하세요.



가상호스트 설정 - vhosts.xml
====================================

서비스할 가상호스트를 설정합니다. 실행파일과 같은 경로에 존재하는 
vhosts.xml파일을 가상호스트 파일로 인식합니다. XML구조는 복수의 가상호스트와 
1개의 기본 가상호스트를 설정하는 구조입니다. ::

    <Vhosts>
        <Vhost Status="Active" Name="web.winesoft.co.kr"> ... </Vhost>
        <Vhost Status="Active" Name="img.winesoft.co.kr"> ... </Vhost>
        <Vhost Status="Active" Name="vod.winesoft.co.kr"> ... </Vhost>
        <Default>web.winesoft.co.kr</Default>
    </Vhosts>
    
가상호스트 설정 따라하기
------------------------------------------------
vhosts.xml을 열고 다음과 같이 편집합니다.

    <Vhosts>
        <Vhost Status="Active" Name="ston.winesoft.co.kr">
        <Origin>
            <Address>ston.winesoft.co.kr</Address>
        </Origin>
    </Vhost>

