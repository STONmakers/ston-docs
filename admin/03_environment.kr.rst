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
    
    접근조건에 따라 접근권한(Role)을 설정할 수 있다. 접근권한이 없는 요청에 대해서는 
    *401 Unauthorized*로 응답한다. Allow조건에 Role을 명시적으로 선언하지 않았을 경우 
    Manager의 Role속성이 적용된다.
    
    - ``Admin`` 모든 API호출이 가능하다.
    - ``User`` Monitoring과 Graph계열 API만 호출할 수 있다.
    - ``Looker`` Graph계열 API만 호출할 수 있다.
    
    서비스포트(80)로 HTTP Method를 호출하는 경우 HttpMethod속성의 영향을 받는다. 
    이 설정이 ON인 경우 서비스포트로 요청되었더라도 HTTP Method라면 ACL의 영향을 받는다. 
    반대로 이 설정이 OFF인 경우 ACL의 영향을 받지 않는다. 
    
    설정파일을 POST(Multipart방식)로 업로드할 때 UploadMultipartName속성의 값을 사용한다.

<Cache>
------------------------------------------------

Cache모듈과 전역자원을 설정합니다. ::

    <Cache>
        <Cleanup>
            <Time>02:00</Time>
            <Age>0</Age>
        </Cleanup>
        <InfoLog Type="size" Unit="1" Retention="5" SysLog="OFF">ON</InfoLog>
        <DenyLog Type="size" Unit="1" Retention="5" SysLog="OFF">ON</DenyLog>
        <OriginErrorLog Type="size" Unit="5" Retention="5" Warning="OFF" SysLog="OFF">ON</OriginErrorLog>
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

