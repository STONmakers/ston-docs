.. _environment:

환경설정
******************

설정은 크게 전역설정(server.xml)과 가상호스트설정(vhosts.xml)으로 나뉩니다. 
그 외의 .txt파일들은 특정 가상호스트의 디테일한 조건들을 설정합니다.

   .. figure:: img/conf_files.jpg
      :align: center

      3개의 .xml파일이 전부입니다.

이 장에서는 각 설정파일과 관계에 대해 설명하며, 가상호스트를 추가하는 방법에 대해서도 알아봅니다.

.. toctree::
   :maxdepth: 2

전역설정 - server.xml
====================================

STON의 기본적인 동작방식을 설정합니다. 실행파일과 같은 경로에 존재하는
server.xml파일을 전역파일로 인식합니다. ::

    <Server>
        <Host> ... </Host>
        <Cache> ... </Cache>
        <VHostDefault> ... </VHostDefault>
        <Https> ... </Https>
    </Server>
    
전역설정은 위와 같이 크게 4영역으로 나뉩니다. 
전역 설정은 단일 XML파일이지만 여기서는 기능별로 나누어서 설명합니다.

<Host> 설정
------------------------------------------------

관리자 정보와 API호출 권한에 설정합니다. ::

    <Host>
        <Name>Machine01</Name>
        <Admin>admin@winesoft.co.kr</Admin>
        <Manager Port="10040" HttpMethod="ON" Role="Admin" UploadMultipartName="confile">
            <Allow>192.168.1.1</Allow>
            <Allow Role="Admin">192.168.2.1-255</Allow>
            <Allow Role="User">192.168.3.0/24</Allow>
            <Allow Role="Looker">192.168.4.0/255.255.255.0</Allow>
        </Manager>
    </Host>


<Cache> 설정
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

