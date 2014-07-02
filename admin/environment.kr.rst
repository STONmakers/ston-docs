.. _env:

기본설정
******************

이 장에서는 설정구조와 분류기준에 대해 설명한다. 
후반부에는 가상호스트 생성과 파괴, 원본서버 주소 설정에 대해 설명한다.
구조를 정확히 이해해야 빠르게 서버를 배치할 수 있을뿐만 아니라 장애상황을 유연하게 극복할 수 있다.

설정은 크게 전역(server.xml)과 가상호스트(vhosts.xml)로 나뉜다.

   .. figure:: img/conf_files.png
      :align: center

      3개의 .xml파일이 전부입니다.

2개의 XML파일로 대부분의 서비스를 구성한다.
여러 TXT파일에는 가상호스트별 예외조건을 설정하는데, 특정기능의 목록을 작성하는데 사용된다.


.. note:
   
   라이센스(license.xml)는 설정이 아니다.


.. toctree::
   :maxdepth: 2


.. _env-global:

전역설정 (server.xml)
====================================

실행파일과 같은 경로에 존재하는 server.xml이 전역설정 파일이다.
XML형식의 텍스트파일이다. ::

    <Server>
        <Host> ... </Host>
        <Cache> ... </Cache>
        <VHostDefault> ... </VHostDefault>
        <Https> ... </Https>
    </Server>
    
여기서는 전역설정의 구조와 간단한 기능위주로 설명한다.
:ref:`access-control` 나 :ref:`snmp` 등 전역설정에 위치하지만 덩치가 큰 기능들에 대해서는 각 주제를 다루는 장에서 설명한다. 


.. _env-host:

<Host>
------------------------------------------------

관리목적의 기능을 설정한다. ::

    <Host>
        <Name>stream_07</Name>
        <Admin>admin@example.com</Admin>
        <Manager Port="10040" HttpMethod="ON" Role="Admin" UploadMultipartName="confile">
            <Allow>192.168.1.1</Allow>
            <Allow Role="Admin">192.168.2.1-255</Allow>
            <Allow Role="User">192.168.3.0/24</Allow>
            <Allow Role="Looker">192.168.4.0/255.255.255.0</Allow>
        </Manager>
    </Host>

-  ``<Name>``
    서버 이름을 설정한다. 
    이름이 입력되지 않으면 시스템 이름이 사용한다.
    
-  ``<Admin>``
    관리자 정보(메일 또는 이름)를 설정한다. 
    이 항목은 SNMP 조회목적으로만 사용된다.
    
-  ``<Manager>``
    관리용도로 사용할 매니저 포트와 ACL(Access Control List)을 설정한다. 
    ACL은 IP, IP범위, BitMask, Subnet 이상 네 가지 형식을 지원한다. 
    접속한 세션이 Allow로 접근이 허가된 IP가 아니면 접속을 차단한다.
    API를 호출하는 IP가 ``<Allow>`` 목록에 반드시 설정되어야 한다.
    
    접근조건에 따라 접근권한(Role)을 설정할 수 있다. 
    접근권한이 없는 요청에 대해서는 **401 Unauthorized** 로 응답한다. 
    ``<Allow>`` 조건에 ``Role`` 속성을 명시적으로 선언하지 않을 경우 ``<Manager>`` 의 ``Role`` 속성이 적용된다.
    
    - ``Admin`` 모든 API호출이 가능하다.
    - ``User`` ref:`api_monitoring` , ref:`api-graph` API만 호출할 수 있다.
    - ``Looker`` ref:`api-graph` API만 호출할 수 있다.
    
    접근권한 이외에 관리목적의 속성을 가진다.
    
    - ``HttpMethod``
    
      - ``ON (기본)`` :ref:`api-etc-httpmethod` 호출시 ACL을 검사한다.
      
      - ``OFF`` :ref:`api-etc-httpmethod` 호출시 ACL을 검사하지 않는다.
    
    - ``UploadMultipartName`` :ref:`api-conf-upload` 의 변수명을 설정한다.


.. _env-cache:

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
        <Listen>0.0.0.0</Listen>        
        <ConfigHistory>30</ConfigHistory>
    </Cache>

-  ``<Cleanup>``
    하루에 한 번 시스템 최적화를 수행한다. 서비스 품질저하를 방지하기 위해 
    최적화는 조금씩 점진적으로 수행된다.

    - ``<Time>`` Cleanup이 수행될 시간을 설정한다. 오후 11시 10분을 설정하고 
    싶다면 23:10으로 설정한다.
    
    - ``<Age> (단위: 일)`` 0보다 큰 경우, 일정 기간동안 한번도 접근되지 않은 콘텐츠를 삭제한다.
    디스크를 미리 확보하여 서비스 시간 중 디스크 부족이 발생하지 않게 한다.
    
-  ``<Storage>``
    콘텐츠를 저장할 디스크를 설정한다. 디스크 개수에 제한은 없다. 각 디스크마다 
    최대 캐싱용량(Quota, 단위: GB)을 설정할 수 있다. Quota를 설정하지 않아도 
    디스크가 꽉 차지 않도록 LRU(Least Recently Used) 알고리즘에 의해 오래된 
    컨텐츠를 자동으로 삭제한다.
    
    디스크는 장애가 가장 많이 발생하는 장비이므로 장애조건을 설정해야 한다.
    DiskFailSec(초)동안 DiskFailCount만큼 디스크 작업이 실패하면 해당 디스크는 자동으로 
    배제된다. 배제된 디스크 상태는 "Invalid"로 제공된다. 모든 디스크가 배제될 수도 있는데
    이 때의 동작방식은 OnCrash속성을 통해 설정 가능하다.
    
    - ``hang`` 장애 디스크를 모두 재투입한다. 정상 서비스를 기대한다기 보다는 복구되기 전까지 트래픽을 버티려는 목적이 강하다.    
    - ``bypass`` 모든 요청을 원본서버으로 바이패스 한다. 디스크가 복구되면 즉시 STON이 서비스를 처리한다.
    - ``selfkill`` STON을 종료시킨다.
    
-  ``<Listen>``
    모든 가상호스트가 Listen할 IP목록을 지정한다. 모든 가상호스트의 기본 Listen설정인 
    *:80은 0.0.0.0:80을 의미한다. 0.0.0.0이 아닌 지정된 IP만을 열고 싶은 경우 다음과 같이 
    Listen할 IP 목록을 명확하게 지정한다. ::

       <Cache>
         <Listen>10.10.10.10</Listen>
         <Listen>10.10.10.11</Listen>
         <Listen>127.0.0.2</Listen>
       </Cache>    

-  ``<ConfigHistory> (단위: 일)``
    STON은 설정이 변경될 때마다 모든 설정을 백업한다. 압축 후 ./conf/ 에 하나의 파일로 저장한다. 
    파일명은 "날짜_시간_HASH.tgz"로 생성된다. ::
    
    20130910_174843_D62CA26F16FE7C66F81D215D8C52266AB70AA5C8.tgz
    
    모든 설정이 완전히 동일하다면 같은 HASH값을 가진다.
    이전 설정으로 Rollback되더라도 새로운 설정으로 저장된다. 
    백업된 설정은 Cleanup시간을 기준으로 ConfigHistory(일) 까지만 저장된다. 
    특별히 ConfigHistory의 제한은 없다.
    
    
.. _env-vhostdefault:
    
<VHostDefault>
------------------------------------------------

관리자는 각각의 가상호스트를 독립적으로 설정할 수 있다. 하지만 가상호스트를 생성할 때마다
동일한 설정을 반복하는 것은 매우 소모적인 일이다.

   .. figure:: img/vhostdefault.png
      :align: center
   
      모든 가상호스트는 <VHostDefault>설정을 상속받는다.

기본적으로 www.example.com과 img.example.com은 <VHostDefault>의 모든 설정을 상속받는다.
www.example.com의 경우 별도로 덮어쓰기(Overriding)한 값이 없으므로 A=1, B=2가 된다. 반면
img.example.com은 B=3으로 덮어쓰기했으므로 A=1, B=3이 된다. 이런 특성을 잘 이해해야 
앞으로 설정이 용이하다.

<VHostDefault>는 기능별로 묶인 5개의 하위 태그(캐싱옵션<Options>, 원본옵션<OriginOptions>, 
미디어<Media>, 통계<Stats>, 로그<Log>)를 가진다. 각 옵션에 대해서는 차차 설명한다. ::

    <VHostDefault>
        <Options> ... </Options>  
        <OriginOptions> ... </OriginOptions>  
        <Media> ... </Media>  
        <Stats> ... </Stats>  
        <Log> ... </Log>
    </VHostDefault>


.. _env-https:

<Https>
------------------------------------------------

HTTPS 서비스를 구성한다. 별도의 장에서 설명한다.



.. _env-vhost:

가상호스트 설정 (vhosts.xml)
====================================

서비스할 가상호스트를 설정한다. 실행파일과 같은 경로에 존재하는 
vhosts.xml파일을 가상호스트 파일로 인식한다. 여러개의 가상호스트를 설정한다. ::

    <Vhosts>
        <Vhost Status="Active" Name="www.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="img.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="vod.example.com"> ... </Vhost>
    </Vhosts>
    
    
.. _env-vhost-create-destroy:
    
가상호스트 생성과 파괴
------------------------------------------------
가상호스트는 vhosts.xml에 <Vhost>태그를 입력하는 것으로 생성된다. ::

    <Vhost Status="Active" Name="ston.example.com">
        <Origin>
            <Address>123.123.123.123</Address>
        </Origin>
    </Vhost>

-  ``<Vhost>`` 가상호스트를 설정한다.
    
    - ``Status (기본: Active)`` Inactive인 경우 해당 가상호스트를 서비스하지 않는다. 캐싱된 콘텐츠는 유지된다.
    - ``Name`` 가상호스트 이름. 반드시 입력되어야 하며 명시적이어야 한다.
    
가상호스를 삭제하려면 해당 가상호스트의 <Vhost>태그를 삭제한다. 삭제된 가상호스트의 
모든 콘텐츠는 삭제대상이 되며 가상호스트를 다시 추가하려도 콘텐츠는 되살아나지 않는다.
그러므로 가상호스트를 신중하게 삭제하려면 일정시간 Status속성을 Inactive상태로 
유지하여 혹시 있을지 모르는 재투입 상황에 대비할 수 있다.


.. _env-vhost-find:
    
가상호스트 찾기
------------------------------------------------
다음은 가장 간단한 형태의 HTTP요청이다. ::

    GET / HTTP/1.1
    Host: www.example.com

일반적인 웹(캐시) 서버는 Host헤더의 값으로 가상호스트를 찾는다. 하나의 가상호스트를
여러 이름으로 서비스하고 싶다면 <Alias>를 사용한다. ::

    <Vhost ...>
        <Alias>www2.example.com</Alias>
        <Alias>*.sub.example.com</Alias>
    </Vhost>

Alias의 개수는 제한이 없다. 명확한 표현(www2.example.com)과 
패턴표현(*.sub.example.com)을 지원한다. 패턴은 복잡한 정규표현식이 아닌 
prefix에 * 표현을 하나만 붙일 수 있는 간단한 형식만을 지원한다.

가상호스트 검색 순서는 다음과 같다.

1. <Vhost>의 Name속성과 일치하는가?
2. <Alias>의 명시적인 이름과 일치하는가?
3. <Alias>의 패턴과 일치하는가?


.. _env-vhost-defaultvhost:    
    
기본 가상호스트
------------------------------------------------
기본 가상호스트를 설정할 수 있다. 클라이언트의 HTTP요청이 가상호스트를 찾지못한 
경우 기본 가상호스트에 의해 처리된다. 반드시 <Vhost>의 Name속성과 똑같은 문자열로 설정한다. ::

    <Vhosts>
        <Vhost Status="Active" Name="www.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="img.example.com"> ... </Vhost>
        <Default>www.example.com</Default>
    </Vhosts>
    
    
.. _env-vhost-listen:
    
서비스 주소
------------------------------------------------
서비스 할 주소와 포트를 설정한다. ::

    <Vhost ...>
        <Listen>*:80</Listen>
    </Vhost>

*:80 표현은 모든 IP의 80포트로 오는 요청을 처리한다는 의미이다. 예를 들어 
특정 IP(1.1.1.1)의 90포트로 서비스하고 싶다면 다음과 같이 설정한다. ::
    
    <Vhost ...>
        <Listen>1.1.1.1:90</Listen>
    </Vhost>
    
서비스 포트를 열지 않으려면 다음과 같이 OFF로 설정한다. ::
    
    <Vhost ...>
        <Listen>OFF</Listen>
    </Vhost>
    

.. _env-vhost-activeorigin:

Active 원본서버
------------------------------------------------
가상호스트는 원본서버를 복제하는 것이 목적이다. 서비스 형태에 맞게 다양한 
원본서버 주소를 설정할 수 있다. ::

    <Vhost ...>
        <Origin>
            <Address>1.1.1.1</Address>
            <Address>1.1.1.2</Address>
        </Origin>
    </Vhost>

주소의 개수는 제한이 없다. Active/Active방식(Round-Robin)으로 선택되며, 
원본서버 주소 포트가 80인 경우 생략할 수 있다. 

예를 들어 다른 포트(8080)로 서비스되는 경우 1.1.1.1:8080과 같이 포트번호를 
명시해야 한다. 주소는 {IP|Domain}{Port}{Path}형식으로 8가지 형식이 가능하다.

============================== ===================
Address                        Host헤더
============================== ===================
1.1.1.1	                       가상호스트명
1.1.1.1:8080	                 가상호스트명:8080       
1.1.1.1/account/dir	           가상호스트명            
1.1.1.1:8080/account/dir       가상호스트명:8080       
example.com	                   example.com             
example.com:8080	             example.com:8080        
example.com/account/dir	       example.com             
example.com:8080/account/dir	 example.com:8080
============================== ===================


예를 들어 원본서버에 example.com/account/dir처럼 경로가 붙어있다면 
요청된 URL은 원본서버 주소 경로 뒤에 붙는다. 클라이언트가 /img.jpg를 요청하면 
최종 주소는 example.com/account/dir/img.jpg가 된다.


.. _env-vhost-standbyorigin:

Standby 원본서버
------------------------------------------------
Standby 원본서버를 설정한다. 주소의 개수는 제한이 없다. ::

    <Vhost ...>
        <Origin>
            <Address>1.1.1.1</Address>
            <Address>1.1.1.2</Address>
            <Address2>1.1.1.3</Address2>
            <Address2>1.1.1.4</Address2>
        </Origin>
    </Vhost>

Standby서버는 모든 Active서버가 정상동작하고 있다면 서비스에 투입되지 않는다. 
Active서버에 장애가 감지되면 해당 서버를 대체하기 위하여 투입되며 Active서버가 
복구되면 다시 Standby상태로 돌아간다. 만약 Standby서버에 장애가 감지되면 
해당 Standby서버가 복구되기 전까지 서비스에 투입되지 않는다.


.. _env-vhost-txt:

그 밖의 예외조건 (.txt)
------------------------------------------------

서비스 중 다음과 같이 예외적인 상황이 필요할 때가 있다.

- 모든 POST요청은 허용하지 않지만, 특정 URL에 대한 POST요청은 허가한다.
- 모든 GET요청은 STON이 응답하지만, 특정 IP대역에 대해서는 원본서버로 바이패스한다.
- 특정 국가에 대해서는 전송속도를 제한한다.

이와같은 예외조건은 .xml에 설정하지 않는다. 모든 가상호스트는 독립적인 예외조건을 가진다.
예외조건은 ./svc/가상호스트/ 디렉토리 하위에 .txt파일형식으로 존재한다.
관련된 설정을 설명할 때 예외조건도 함께 다룬다.


