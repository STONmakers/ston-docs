.. _env:

3장. 설정구조
******************

이 장에서는 설정구조와 변경된 설정을 적용하는 방법에 대해 설명한다.
구조를 정확히 이해해야 빠르게 서버를 배치할 수 있을뿐만 아니라 장애상황을 유연하게 극복할 수 있다.

설정은 크게 전역(server.xml)과 가상호스트(vhosts.xml)로 나뉜다.

   .. figure:: img/conf_files.png
      :align: center

      2개의 .xml파일이 전부입니다.

2개의 XML파일로 대부분의 서비스를 구성한다.
여러 TXT파일에는 가상호스트별 예외조건을 설정하는데, 특정기능의 목록을 작성하는데 사용된다.
기능설명을 위해 다음처럼 완전한 형태의 XML을 예시하는 것은 굉장히 번거롭다. ::

   <Server>
       <VHostDefault>
           <Options>
               <CaseSensitive>ON</CaseSensitive>
           </Options>
       </VHostDefault>
   </Server>

때문에 다음과 같이 축약하여 설명한다. ::

   # server.xml - <Server><VHostDefault><Options>

   <CaseSensitive>ON</CaseSensitive>


.. note::

   라이센스(license.xml)는 설정이 아니다.


.. _api-conf-reload:

설정 Reload
====================================

설정 변경 후 관리자가 명확하게 API를 호출해야 한다.
시스템과 성능 관련설정을 제외한 대부분의 설정은 서비스 중단없이 즉시 적용된다. ::

   http://127.0.0.1:10040/conf/reload

설정이 변경될 때마다 :ref:`admin-log-info` 에 변경사항이 기록된다.


server.xml 전역설정
====================================

실행파일과 같은 경로에 존재하는 server.xml이 전역설정 파일이다.
XML형식의 텍스트파일이다. ::

    # server.xml

    <Server>
        <Host> ... </Host>
        <Cache> ... </Cache>
        <VHostDefault> ... </VHostDefault>
    </Server>

우선 전역설정의 구조와 간단한 기능위주로 설명한다.
:ref:`access-control` 나 :ref:`snmp` 등 전역설정에 위치하지만 덩치가 큰 기능들에 대해서는 각 주제를 다루는 장에서 설명한다.

.. toctree::
   :maxdepth: 2


.. _env-host:

관리자 설정
------------------------------------

관리목적의 기능을 설정한다. ::

    # server.xml - <Server>

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
    - ``User`` :ref:`api_monitoring` , :ref:`api-graph` API만 호출할 수 있다.
    - ``Looker`` :ref:`api-graph` API만 호출할 수 있다.

    기타 다음과 같은 자잘한 관리목적의 속성을 가진다.

    - ``HttpMethod``

      - ``ON (기본)`` :ref:`api-etc-httpmethod` 호출시 ACL을 검사한다.

      - ``OFF`` :ref:`api-etc-httpmethod` 호출시 ACL을 검사하지 않는다.

    - ``UploadMultipartName`` :ref:`api-conf-upload` 의 변수명을 설정한다.


.. _env-cache-storage:

Storage 구성
------------------------------------

Caching된 콘텐츠를 저장할 Storage를 구성한다. ::

    # server.xml - <Server>

    <Cache>
        <Storage DiskFailSec="60" DiskFailCount="10" OnCrash="hang">
            <Disk>/user/cache1</Disk>
            <Disk>/user/cache2</Disk>
            <Disk Quota="100">/user/cache3</Disk>
        </Storage>
    </Cache>

-  ``<Storage>``
    콘텐츠를 저장할 디스크를 설정한다.
    하위 ``<Disk>`` 개수제한은 없다.

    디스크는 장애가 가장 많이 발생하는 장비이기 때문에 명확한 장애조건을 설정할 것을 권장한다.
    ``DiskFailSec (기본: 60초)`` 동안 ``DiskFailCount (기본: 10)`` 만큼 디스크 작업이 실패하면
    해당 디스크는 자동으로 배제된다.
    배제된 디스크 상태는 "Invalid"로 명시된다.

    모든 디스크가 배제될 수도 있는데 이 때의 동작방식은 ``OnCrash`` 속성으로 설정한다.

    - ``hang (기본)`` 장애 디스크를 모두 재투입한다.
      정상 서비스를 기대한다기 보다는 원본을 보호하려는 목적이 강하다.

    - ``bypass`` 모든 요청을 원본서버로 바이패스 한다.
      디스크가 복구되면 즉시 STON이 서비스를 처리한다.

    - ``selfkill`` STON을 종료시킨다.

각 디스크마다 최대 캐싱용량을 ``Quota (단위: GB)`` 속성으로 설정할 수 있다.
굳이 설정하지 않더라도 항상 디스크가 꽉 차지 않도록 LRU(Least Recently Used) 알고리즘에 의해 오래된 콘텐츠를 자동으로 삭제한다.
특별히 호환성에 문제가 있는 파일시스템은 없다. 그러므로 관리자가 친숙한 파일 시스템을 사용해도 성능에 큰 영향은 없다.




.. _env-cache-resource:

메모리 제한
------------------------------------

사용할 최대 메모리와 컨텐츠 적재비율 설정한다. ::

    # server.xml - <Server>

    <Cache>
        <SystemMemoryRatio>100</SystemMemoryRatio>
        <ContentMemoryRatio>50</ContentMemoryRatio>
    </Cache>

-  ``<SystemMemoryRatio> (기본: 100%)``

   시스템 메모리에서 STON이 사용할 최대 메모리를 비율로 설정한다.
   예를 들어 16GB장비에서 이 수치를 50(%)으로 설정하면 시스템 메모리가 8GB인 것처럼 동작한다.
   특히 :ref:`filesystem` 등을 통해 다른 프로세스와 연동할 때 유용하다.

-  ``<ContentMemoryRatio> (기본: 50%)``

   STON은 디스크에서 로딩된 Body 데이터를 메모리에 최대한 Caching하여 서비스 품질을 향상시킨다.
   서비스 형태에 따라 이 비율을 조절하여 품질을 최적화한다.

      .. figure:: img/bodyratio1.png
         :align: center

         ContentMemoryRatio를 통해 메모리비율을 설정한다.

   예를 들어 게임 다운로드처럼 파일 개수는 많지 않지만 Contents크기가 큰 서비스의 경우 File I/O 부하가 부담스럽다.
   이런 경우 ``<ContentMemoryRatio>`` 를 높여서 보다 많은 Contents데이터가 메모리에 상주할 수 있도록 설정하면 서비스 품질을 높일 수 있다.

      .. figure:: img/bodyratio2.png
         :align: center

         ContentMemoryRatio를 높이면 I/O가 감소한다.



기타 Caching 설정
------------------------------------

기타 Caching서비스의 기반동작을 설정한다. ::

    # server.xml - <Server>

    <Cache>
        <Cleanup>
            <Time>02:00</Time>
            <Age>0</Age>
        </Cleanup>
        <Listen>0.0.0.0</Listen>
        <ConfigHistory>30</ConfigHistory>
    </Cache>

-  ``<Cleanup>``
    하루에 한 번 시스템 최적화를 수행한다.
    최적화의 대부분은 디스크정리 작업으로 I/O 부하가 발생한다.
    서비스 품질저하를 방지하기 위해 최적화는 조금씩 점진적으로 수행된다.

    - ``<Time> (기본: AM 2)`` Cleanup 수행시간을 설정한다. 오후 11시 10분을 설정하고 싶다면 23:10으로 설정한다.

    - ``<Age> (기본: 0, 단위: 일)`` 0보다 큰 경우, 일정 기간동안 한번도 접근되지 않은 콘텐츠를 삭제한다.
      디스크를 미리 확보하여 서비스 시간 중 디스크 부족이 발생할 확률을 줄이기 위함이다.

-  ``<Listen>``
    모든 가상호스트가 Listen할 IP목록을 지정한다.
    모든 가상호스트의 기본 Listen설정인 *:80은 0.0.0.0:80을 의미한다.
    지정된 IP만을 열고 싶은 경우 다음과 같이 명확하게 설정한다. ::

       # server.xml - <Server>

       <Cache>
         <Listen>10.10.10.10</Listen>
         <Listen>10.10.10.11</Listen>
         <Listen>127.0.0.2</Listen>
       </Cache>

-  ``<ConfigHistory> (기본: 30일)``
    STON은 설정이 변경될 때마다 모든 설정을 백업한다.
    압축 후 ./conf/ 에 하나의 파일로 저장된다.
    파일명은 "날짜_시간_HASH.tgz"로 생성된다. ::

       20130910_174843_D62CA26F16FE7C66F81D215D8C52266AB70AA5C8.tgz

    모든 설정이 완전히 동일하다면 같은 HASH값을 가진다.
    :ref:`api-conf-restore` 가 호출되도 새로운 설정으로 저장된다.
    백업된 설정은 Cleanup시간을 기준으로 설정된 날만큼만 저장된다.
    설정파일 저장의 날짜제한은 없다.



강제 Cleanup
------------------------------------

API호출로 Cleanup한다. ``<Age>`` 를 파라미터로 입력할 수 있다. ::

   http://127.0.0.1:10040/command/cleanup
   http://127.0.0.1:10040/command/cleanup?age=10

``<Age>`` 가 0이라면 디스크 공간이 부족하다고 판단될 때만 Cleanup을 수행한다.
``<Age>`` 파라미터가 0보다 크다면 해당 "일"동안 한번도 접근되지 않은 콘텐츠를 삭제한다.


.. _env-vhostdefault:

가상호스트 기본설정
------------------------------------

관리자는 각각의 가상호스트를 독립적으로 설정할 수 있다.
하지만 가상호스트를 생성할 때마다 동일한 설정을 반복하는 것은 매우 소모적이다.
모든 가상호스트는 ``<VHostDefault>`` 을 상속받는다.

   .. figure:: img/vhostdefault.png
      :align: center

      단일 상속이다.

www.example.com의 경우 별도로 덮어쓰기(Overriding)한 값이 없으므로 A=1, B=2가 된다.
반면 img.example.com은 B=3으로 덮어쓰기했으므로 A=1, B=3이 된다.
관리자들은 보통 같은 서비스특성을 가지는 서비스를 한 서버에 같이 구성한다.
그러므로 상속은 매우 효과적인 방법이다.

``<VHostDefault>`` 는 기능별로 묶인 5개의 하위 태그를 가진다. ::

    # server.xml - <Server>

    <VHostDefault>
        <Options> ... </Options>
        <OriginOptions> ... </OriginOptions>
        <Media> ... </Media>
        <Stats> ... </Stats>
        <Log> ... </Log>
    </VHostDefault>

예를 들어 :ref:`media` 기능은 ``<Media>`` 하위에 구성하는 식이다.


.. _env-vhost:

vhosts.xml 가상호스트 설정
====================================

실행파일과 같은 경로에 존재하는 vhosts.xml파일을 가상호스트 설정파일로 인식한다.
가상호스트 개수에 제한은 없다. ::

    # vhosts.xml

    <Vhosts>
        <Vhost Status="Active" Name="www.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="img.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="vod.example.com"> ... </Vhost>
    </Vhosts>


.. _env-vhost-create-destroy:

생성/파괴
------------------------------------

``<Vhosts>`` 하위에 ``<Vhost>`` 로 가상호스트를 설정한다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Status="Active" Name="www.example.com">
        <Origin>
            <Address>10.10.10.10</Address>
        </Origin>
    </Vhost>

-  ``<Vhost>`` 가상호스트를 설정한다.

   - ``Status (기본: Active)`` Inactive인 경우 해당 가상호스트를 서비스하지 않는다. 캐싱된 콘텐츠는 유지된다.
   - ``Name`` 가상호스트 이름. 중복될 수 없다.

``<Vhost>`` 를 삭제하면 해당 가상호스트가 삭제된다.
삭제된 가상호스트의 모든 콘텐츠는 삭제대상이 된다.
다시 추가해도 콘텐츠는 되살아나지 않는다.


.. _env-vhost-find:

찾기
------------------------------------

다음은 가장 간단한 형태의 HTTP요청이다. ::

    GET / HTTP/1.1
    Host: www.example.com

일반적인 Web서버는 Host헤더로 가상호스트를 찾는다.
하나의 가상호스트를 여러 이름으로 서비스하고 싶다면 ``<Alias>`` 를 사용한다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="example.com">
        <Alias>another.com</Alias>
        <Alias>*.sub.example.com</Alias>
    </Vhost>

-  ``<Alias>``

   가상호스트의 별명을 설정한다.
   개수는 제한이 없다.
   명확한 표현(another.com)과 패턴표현(*.sub.example.com)을 지원한다.
   패턴은 복잡한 정규표현식이 아닌 prefix에 * 표현을 하나만 붙일 수 있는 간단한 형식만을 지원한다.


가상호스트 검색 순서는 다음과 같다.

1. ``<Vhost>`` 의 ``Name`` 과 일치하는가?
2. 명시적인 ``<Alias>`` 와 일치하는가?
3. 패턴 ``<Alias>`` 를 만족하는가?


.. _env-vhost-defaultvhost:

Default 가상호스트
------------------------------------

요청을 처리할 가상호스트를 찾지못한 경우 선택될 가상호스트를 지정할 수 있다.
요청을 처리하고 싶지 않다면 설정하지 않아도 된다. ::

    # vhosts.xml

    <Vhosts>
        <Vhost Status="Active" Name="www.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="img.example.com"> ... </Vhost>
        <Default>www.example.com</Default>
    </Vhosts>

-  ``<Default>``

   기본 가상호스트 이름을 설정한다.
   반드시 ``<Vhost>`` 의 ``Name`` 속성과 똑같은 문자열로 설정해야 한다.


.. _env-vhost-listen:

서비스주소
------------------------------------
서비스 주소를 설정한다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="www.example.com">
        <Listen>*:80</Listen>
    </Vhost>

-  ``<Listen> (기본: *:80)``

   {IP}:{Port} 형식으로 서비스 주소를 설정한다.
   *:80 표현은 모든 NIC로부터의 80포트로 오는 요청을 처리한다는 의미다.
   예를 들어 특정 IP(1.1.1.1)의 90포트로 서비스하고 싶다면 다음과 같이 설정한다. ::

       # vhosts.xml - <Vhosts>

       <Vhost Name="www.example.com">
           <Listen>1.1.1.1:90</Listen>
       </Vhost>

.. note:

   서비스 포트를 열지 않으려면 ``OFF`` 로 설정한다. ::

      # vhosts.xml - <Vhosts>

      <Vhost Name="www.example.com">
         <Listen>OFF</Listen>
      </Vhost>


.. _env-vhost-txt:

가상호스트-예외조건 (.txt)
---------------------------------------

서비스 중 다음과 같이 예외적인 상황이 필요할 때가 있다.

- 모든 POST요청은 허용하지 않지만, 특정 URL에 대한 POST요청은 허가한다.
- 모든 GET요청은 STON이 응답하지만, 특정 IP대역에 대해서는 원본서버로 바이패스한다.
- 특정 국가에 대해서는 전송속도를 제한한다.

이와같은 예외조건은 XML에 설정하지 않는다.
모든 가상호스트는 독립적인 예외조건을 가진다.
예외조건은 ./svc/가상호스트/ 디렉토리 하위에 TXT로 존재한다.
관련 기능에 대해 설명할 때 예외조건도 함께 다룬다.


가상호스트 목록확인
====================================

가상호스트 목록을 조회한다. ::

   http://127.0.0.1:10040/monitoring/vhostslist

결과는 JSON형식으로 제공된다. ::

   {
      "version": "1.1.9",
      "method": "vhostslist",
      "status": "OK",
      "result": [ "www.example.com","www.foobar.com", "site1.com" ]
   }


.. _api-conf-show:

설정 확인
====================================

서비스 중인 설정파일을 확인한다.
txt파일들은 가상호스트(vhost)를 명확하게 지정해주어야 한다. ::

    http://127.0.0.1:10040/conf/server.xml
    http://127.0.0.1:10040/conf/vhosts.xml
    http://127.0.0.1:10040/conf/querystring.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/bypass.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/ttl.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/expires.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/acl.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/headers.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/throttling.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/postbody.txt?vhost=www.example.com


.. _api-conf-history:

설정 목록
====================================

백업된 설정목록을 열람한다. ::

    http://127.0.0.1:10040/conf/latest
    http://127.0.0.1:10040/conf/history

결과는 JSON 형식으로 제공된다.
빠르게 마지막 설정상태만 확인하고 싶은 경우는 /conf/latest를 사용할 것을 권장한다. ::

    {
        "history" :
        [
            {
                "id" : "5",
                "conf-date" : "2013-11-06",
                "conf-time" : "15:26:37",
                "type" : "loaded",
                "size" : "16368",
                "hash" : "D62CA26F16FE7C66F81D215D8C52266AB70AA5C8",
                "ver": "1.2.8"
            },
            {
                "id" : "6",
                "conf-date" : "2013-11-07",
                "conf-time" : "07:02:21",
                "type" : "modified",
                "size" : "27544",
                "hash" : "F81D215D8C52266AB70AA5C8D62CA26F16FE7C66",
                "ver": "1.2.8"
            }
        ]
    }

-  ``id`` 설정의 고유 아이디 (Reload할때마다 +1)
-  ``conf-date`` 설정 변경날짜
-  ``conf-time`` 설정 변경시간
-  ``type`` 설정이 반영된 형태
   - ``loaded`` STON이 시작될 때
   - ``modified`` 설정이 (관리자 또는 WM에 의해) 변경될 때
   - ``uploaded`` 설정파일 API를 통해 업로드 되었을 때
   - ``restored`` 설정파일이 API를 통해 복구되었을 때
-  ``size`` 설정파일 크기
-  ``hash`` 설정파일을 SHA-1으로 hash한 값


.. _api-conf-restore:

설정 복구
====================================

hash값 또는 id를 기준으로 원하는 시점의 설정으로 되돌린다.
hash와 id가 모두 명시된 경우 hash값이 우선한다.
정상적으로 Rollback된 경우 200 OK, 실패한 경우 500 Internal Error로 응답한다. ::

    http://127.0.0.1:10040/conf/restore?hash=...
    http://127.0.0.1:10040/conf/restore?id=...


.. _api-conf-download:

설정 다운로드
====================================

hash값 또는 id를 기준으로 원하는 시점의 설정을 다운로드 한다.
Content-Type은 "application/x-compressed"로 명시된다.
hash와 id가 모두 명시된 경우 hash값이 우선하며, 해당 시점의 설정이 존재하지 않는 경우
404 NOT FOUND로 응답한다. ::

    http://127.0.0.1:10040/conf/download?hash=...
    http://127.0.0.1:10040/conf/download?id=...

.. _api-conf-upload:

설정 업로드
====================================

설정파일을 HTTP Post방식(Multipart 지원)으로 업로드 한다. ::

    http://127.0.0.1:10040/conf/upload

다음과 같이 주소, Content-Length, Content-Type(="multipart/form-data")이 명확하게 선언되어 있어야 한다. ::

    POST /conf/upload
    Content-Length: 16455
    Content-Type: multipart/form-data; boundary=......

업로드가 완료되면 압축을 해지한 뒤 즉시 반영시킨다.

Multipart방식에서는 "confile"을 기본 이름으로 사용한다.
이 값은 ``<Manager>`` 의 ``UploadMultipartName`` 속성에서 설정할 수 있다. ::

    <form enctype="multipart/form-data" action="http://127.0.0.1:10040/conf/upload" method="POST">
        <input name="confile" type="file" />
        <input type="submit" value="Upload" />
    </form>
