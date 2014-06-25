.. _api_conf:

/Conf
******************

설정파일 관련 API목록.

.. toctree::
   :maxdepth: 2

.. _show:

설정파일 확인
====================================

서비스 중인 설정파일을 확인한다. 
txt파일들은 가상호스트(vhost)를 명확하게 지정해주어야 한다. ::

    http://127.0.0.1:10040/conf/server.xml
    http://127.0.0.1:10040/conf/vhosts.xml
    http://127.0.0.1:10040/conf/querystring.txt?vhost=www.site1.com
    http://127.0.0.1:10040/conf/bypass.txt?vhost=www.site1.com
    http://127.0.0.1:10040/conf/ttl.txt?vhost=www.site1.com
    http://127.0.0.1:10040/conf/expires.txt?vhost=www.site1.com
    http://127.0.0.1:10040/conf/acl.txt?vhost=www.site1.com
    http://127.0.0.1:10040/conf/headers.txt?vhost=www.site1.com
    http://127.0.0.1:10040/conf/throttling.txt?vhost=www.site1.com
    http://127.0.0.1:10040/conf/postbody.txt?vhost=www.site1.com


.. _reload:

설정 Reload
====================================

서비스 중단없이 변경된 설정을 적용한다. 
동적으로 가상호스트의 추가, 삭제, 변경이 가능하다. ::

    http://127.0.0.1:10040/conf/reload
    
일부 설정은 반드시 재시작해야 반영된다.


.. _list:

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


.. _restore:

설정 Restore
====================================

hash값 또는 id를 기준으로 원하는 시점의 설정으로 되돌린다. 
hash와 id가 모두 명시된 경우 hash값이 우선한다. 
정상적으로 Rollback된 경우 200 OK, 실패한 경우 500 Internal Error로 응답한다. ::

    http://127.0.0.1:10040/conf/restore?hash=...
    http://127.0.0.1:10040/conf/restore?id=...


.. _download:
    
설정 다운로드
====================================

hash값 또는 id를 기준으로 원하는 시점의 설정을 다운로드 한다. 
Content-Type은 "application/x-compressed"로 명시된다. 
hash와 id가 모두 명시된 경우 hash값이 우선하며, 해당 시점의 설정이 존재하지 않는 경우 
404 NOT FOUND로 응답한다. ::

    http://127.0.0.1:10040/conf/download?hash=...
    http://127.0.0.1:10040/conf/download?id=...

.. _upload:

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
    
    
