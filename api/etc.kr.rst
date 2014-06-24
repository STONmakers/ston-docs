.. _api_etc:

기타
******************

기타 API.

.. toctree::
   :maxdepth: 2


Help
====================================

API 목록을 확인한다.

    http://127.0.0.1:10040/help


Version
====================================

버전(WineSOFT STON [Version 2.0.0])을 확인한다.

    http://127.0.0.1:10040/version


HTTP Method 지원
====================================

Purge, Expire, ExpireAfter, HardPurge의 경우 HTTP Method로 지원된다. 
다음과 같은 HTTP 규격으로 API를 호출할 수 있다. ::

    PURGE /sample.dat HTTP/1.1
    host: ston.winesoft.co.kr
    
HTTP Method는 기본적으로 Manager포트와 서비스(80)포트에서 동작한다. 
서비스포트로 요청되는 HTTP Method의 ACL은 Manager설정을 참고한다.


POST 지원
====================================

Purge, Expire, ExpireAfter, HardPurge의 경우 POST Method로 호출하는 URI가 지원된다. 
다음과 같은 HTTP 규격으로 API를 호출한다. ::

    POST /command/purge HTTP/1.1
    Content-Length: 37
 
    url=http://ston.winesoft.co.kr/sample.dat