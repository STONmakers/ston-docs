.. _api_monitoring:

/Monitoring
******************

모니터링 관련 API목록.

.. toctree::
   :maxdepth: 2



.. _api-monitoring-vhostlist:
   
가상호스트 목록조회
====================================

가상호스트 목록을 조회한다. ::

    http://127.0.0.1:10040/monitoring/vhostslist
    
결과는 JSON형식으로 제공된다. ::

    {
        "version": "2.0.0",
        "method": "vhostslist",
        "status": "OK",
        "result": [ "www.example.com","www.winesoft.com", "site1.com" ] 
    }



.. _api-monitoring-stats:
   
통계
====================================

언제나 서비스를 모니터링할 수 있다. 
STON은 실시간(Realtime), 5분 평균 통계를 JSON(Default)과 XML형식으로 제공한다. ::

    http://127.0.0.1:10040/monitoring/realtime?type=[JSON 또는 XML]
    http://127.0.0.1:10040/monitoring/average?type=[JSON 또는 XML]
    
-  ``realtime`` 1초 전 서비스 상태를 제공한다.

-  ``average`` 5분 단위 통계를 제공한다.



.. _api-monitoring-fileinfo:
   
캐싱정보
====================================

캐싱하고 있는 파일상태를 모니터링한다. 
일반적으로 파일은 URL로 구분되지만 같은 URL에 다른 옵션(i.e. Accept-Encoding등)이 
존재하는 경우 여러 개의 파일이 존재할 수 있다. ::

    http://127.0.0.1:10040/monitoring/fileinfo?url=example.com/sample.dat
    
결과는 JSON형식으로 제공된다.
다음은 /sample.dat파일의 정보를 열람한 결과이다. ::

    {
        "version": "2.0.0",
        "method": "fileinfo",
        "status": "OK",
        "result":
        [ 
            {
                "URI": "/sample.dat",
                "Accept-Encoding": "N",
                "RefCount": 0,
                "Disk-Index": 0,
                "Size": 2100267,
                "FID": 24267,
                "LocalPath": "/cache1/example.com/000i/q3.bin",
                "File-Opened ": "N",
                "File-Updating": "-",
                "Downloader-Count": "0",
                "LastAccess": [ 2012.09.03 14:29:50, -2 ],
                "UpdateTime": [ 2012.09.03 13:53:43, -2169 ],
                "TTL-Left": [ 2012.10.03 13:53:43, 2589831 ],
                "ResponseCode": 200,
                "ContentType": "text/plain",
                "LastModifiedTime": [ 2010.11.22 20:31:47, -56224685 ],
                "ExpireTime": [ 0, 0 ],
                "CacheControl": "not-specified",
                "ETag": "502dd614:200c2b",
                "CustomTTL": 0,
                "NoMoreExist": "N",
                "LocalFileExist": "Y",
                "SmallFile": "N",
                "State": "Cached",
                "Deleted": "N",
                "AddedSize": "Y",
                "TransferEncoding": "N",
                "Compression": "-",
                "Purge": "N",
                "Ignore-IMS ": "N",
                "Redirect-Location ": "-",
                "Content-Disposition ": "-",
                "NoCache": "N"
            }
        ]
    }
    
-  ``URI`` 파일 URI
-  ``Accept-Encoding`` ("Y" or "N") Accept-Encoding을 지원한다면 "Y"
-  ``RefCount`` 파일참조 카운트
-  ``Size`` (Bytes) 파일크기
-  ``Disk-Index`` (0부터 시작) 저장된 디스크 인덱스
-  ``FID`` 파일 ID
-  ``LocalPath`` 로컬 경로
-  ``File-Opened`` ("Y" or "N") 로컬파일을 열고 있다면 "Y"
-  ``File-Updating`` 파일을 갱신 중이라면 갱신하는 객체의 포인터가 명시
-  ``Downloader-Count`` 원본서버에서 이 파일을 다운로드 받는 현재 세션의 개수
-  ``LastAccess`` (마지막 접근시간, 마지막 접근시간-현재시간) [ 2012.09.03 14:29:50, -2 ]의 의미는 2012.09.03 14:29:50에 접근됐으며 현재로부터 2초 전에 접근됐다는 의미이다.
-  ``UpdateTime`` (갱신시간, 갱신시간-현재시간) 파일이 마지막으로 갱신된 시간. 304 Not Modified에도 시간은 갱신된다.
-  ``TTL-Left`` (만료시간, 만료시간-현재시간) 컨텐츠 만료 예정시간. TTL이 남았다면 양수로, 만료됐다면 음수로 표기된다.
-  ``ResponseCode`` 원본서버 응답코드
-  ``ContentType`` MIME Type
-  ``LastModifiedTime`` (Last Modified Time, Last Modified Time`` 현재시간) 원본서버가 보낸 Last Modified Time. 원본서버가 이 값을 보내지 않았다면 0으로 표시된다.
-  ``ExpireTime`` (Expire Time, Expire Time`` 현재시간) 원본서버가 보낸 Expire Time. 원본서버가 이 값을 보내지 않았다면 0으로 표시된다.
-  ``CacheControl`` ("no-cache" or "not-specified" or (정수)) 원본서버가 보낸 Cache-Contorl 값
-  ``ETag`` STON이 생성한 ETag
-  ``CustomTTL`` 커스텀 TTL. 설정되어 있지 않다면 0이다.
-  ``NoMoreExist`` ("Y" or "N") 파일을 파기예약되어 있다면 "Y"
-  ``LocalFileExist`` ("Y" or "N") 로컬에 파일이 존재하면 "Y" (200 OK가 아닌 파일들은 항상 "Y")
-  ``SmallFile`` ("Y" or "N") 파일을 작은파일로 판단한다면 "Y" (개발적인 이유)
-  ``State`` ("Not Init" or "Cached" or "Error") 파일 상태
-  ``Deleted`` ("Y" or "N") 삭제되었다면 "Y" (개발적인 이유)
-  ``AddedSize`` ("Y" or "N") 크기가 통계에 반영되었다면 "Y" (개발적인 이유)
-  ``TransferEncoding`` ("Y" or "N") Transfer-Encoding을 지원한다면 "Y"
-  ``Compression`` 압축방식
-  ``Purge`` ("Y" or "N") Purge됐다면 "Y"
-  ``Ignore-IMS`` ("Y" or "N") 갱신할 때 If-Modified-Since헤더를 보내지 않도록 설정되었다면 "Y"
-  ``Redirect-Location`` Location 헤더 값
-  ``Content-Disposition`` Content-Disposition 헤더 값
-  ``NoCache`` ("Y" or "N") 원본서버에서 no-cache응답을 줬다면 "Y"



.. _api-monitoring-logtrace:
   
Log Trace
====================================

기록되는 로그를 실시간으로 받아본다. 
Access, Origin, Monitoing로그는 가상호스트(vhost)를 지정해야 한다. ::

    http://127.0.0.1:10040/monitoring/logtrace/info
    http://127.0.0.1:10040/monitoring/logtrace/deny
    http://127.0.0.1:10040/monitoring/logtrace/sys
    http://127.0.0.1:10040/monitoring/logtrace/originerror
    http://127.0.0.1:10040/monitoring/logtrace/access?vhost=www.site1.com
    http://127.0.0.1:10040/monitoring/logtrace/origin?vhost=www.site1.com
    http://127.0.0.1:10040/monitoring/logtrace/monitoring?vhost=www.site1.com



.. _api-monitoring-dnslist:
   
Domain Resolving List
====================================

원본서버 주소를 Domain으로 설정한 경우 Resolving결과를 저장한다. 
최신 Domain 결과를 반영하기위해 최소 1초 ~ 최대 10초 단위로 Domain을 갱신한다. 
Resolving결과가 변경되었다면 즉시 서비스에 반영한다. ::

    http://127.0.0.1:10040/monitoring/dnslist
    
결과는 JSON형식으로 제공된다. ::

    {
        "dns" : 
        [
            {
                "domain" : "www.winesoft.co.kr", 
                "updated-date" : "2013-02-10", 
                "updated-time" : "09:23:11", 
                "checked-date" : "2013-02-15", 
                "checked-time" : "10:55:16", 
                "ip-addresses" : [ "10.10.10.10", "10.10.10.11" ],
                "backup-ip-addresses" : [ "10.10.10.10", "10.10.10.11", "10.10.10.12", "10.10.10.13", "10.10.10.14" ]
            },
            {
                "domain" : "example.com", 
                "updated-date" : "2013-02-14", 
                "updated-time" : "03:09:00", 
                "checked-date" : "2013-02-15", 
                "checked-time" : "10:55:18", 
                "ip-addresses" : [ "20.20.20.20" ]
                "backup-ip-addresses" : [ "20.20.20.20", "20.20.20.11", "20.20.20.12", "20.20.20.13" ]
            },
        ]
    }

-  ``domain`` 도메인 이름
-  ``updated-date`` Resolving결과를 캐싱한 날짜
-  ``updated-time`` Resolving결과를 캐싱한 시간
-  ``checked-date`` 마지막으로 Resolving한 날짜
-  ``checked-time`` 마지막으로 Resolving한 시간
-  ``ip-addresses`` IP 리스트
-  ``backup-ip-addresses`` 백업된 IP 리스트 (Resolving이 실패하면 사용됨)



.. _api-monitoring-hwinfo:
   
하드웨어 정보조회
====================================

하드웨어 정보를 조회한다. ::

    http://127.0.0.1:10040/monitoring/hwinfo
    
결과는 JSON형식으로 제공된다. ::

    {
        "version": "2.0.0",
        "method": "hwinfo",
        "status": "OK",
        "result":
        {
            "OS" : "Linux version 3.3.0 ...(생략)...", 
            "STON" : "1.1.9", 
            "CPU" : 
            { 
                "ProcCount": "4", 
                "Model": "Intel(R) Xeon(R) CPU           E5606  @ 2.13GHz", 
                "MHz": "1200.000", 
                "Cache": "8192 KB"
            }, 
            "Memory" : "8 GB", 
            "NIC" : 
            [ 
                { 
                    "Dev" : "eth1", 
                    "Model" : "Intel Corporation 82574L Gigabit Network Connection", 
                    "IP" : "192.168.0.13", 
                    "MAC" : "00:25:90:36:f4:cb" 
                } 
            ], 
            "Disk" : 
            [ 
                { 
                    "Dev" : "sda", 
                    "Model" : "HP DG0146FAMWL (scsi)", 
                    "Total" : "238787584", 
                    "Usage" : "40181760" 
                } ,
                {
                    "Dev" : "sdb", 
                    "Model" : "HITACHI HUC103014CSS600 (scsi)", 
                    "Total" : "144706478080", 
                    "Usage" : "2101075968" 
                } ,
                {
                    "Dev" : "sdc", 
                    "Model" : "HITACHI HUC103014CSS600 (scsi)", 
                    "Total" : "144706478080", 
                    "Usage" : "2012160000" 
                }
            ]
        } 
    }
    
    

.. _api-monitoring-ciphersuite:
   
CipherSuite 조회
====================================

HTTPS의 CipherSuite설정결과를 조회한다. 
CipherSuite표현식은 `OpenSSL 1.0.0E <http://www.openssl.org/docs/apps/ciphers.html>`_ 를 준수한다. ::

    http://127.0.0.1:10040/monitoring/ssl?ciphersuite=...
    
결과는 JSON형식으로 제공된다. ::

    {
        "version": "2.0.0",
        "method": "ssl",
        "status": "OK",
        "result":
        [
            {
                "Name" : "AES128-SHA", 
                "Ver" : "SSLv3", 
                "Kx" : "RSA", 
                "Au" : "RSA", 
                "Enc" : "AES(128)", 
                "Mac" : "SHA1"
            },
            {
                "Name" : "AES256-SHA", 
                "Ver" : "SSLv3", 
                "Kx" : "RSA", 
                "Au" : "RSA", 
                "Enc" : "AES(256)", 
                "Mac" : "SHA1"
            }
        ]
    }
    
    

.. _api-monitoring-geoiplist:
   
GeoIP파일목록 조회
====================================

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
