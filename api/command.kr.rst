.. _api-cmd:

/Command
******************

/command 계열 API는 STON을 제어하는 역할을 한다.

.. toctree::
   :maxdepth: 2

.. _api-cmd-purge:

Purge
====================================

타겟 컨텐츠를 무효화시켜 원본서버로부터 컨텐츠를 다시 다운로드 받도록 한다. 
Purge후 최초 접근 시점에 원본서버로부터 컨텐츠를 다시 캐싱한다. 
만약 원본서버에 장애가 발생하여 컨텐츠를 가져올 수 없다면 무효화된 컨텐츠를 다시 복원시켜 
서비스에 장애가 없도록 처리한다. 
이렇게 복원된 컨텐츠는 해당 시점으로부터 ConnectTimeout설정만큼 뒤에 갱신한다. ::

    http://127.0.0.1:10040/command/purge?url=...
    
타겟 컨텐츠는 URL, 디렉토리, 패턴으로 지정할 수 있을 뿐만 아니라 "|"(Vertical Bar)를 
구분자를 사용하여 복수의 도메인에 복수의 타겟을 지정할 수 있다. 
만약 도메인 이름이 생략되었다면 최근 사용된 도메인을 사용한다. ::

    http://127.0.0.1:10040/command/purge?url=http://www.site1.com/image.jpg
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image.jpg
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image/bmp/
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image/*.bmp
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image1.jpg|/css/style.css|/script/script.js
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image1.jpg|www.site2.com/page/*.html
    
결과는 JSON형식으로 제공된다. 
타겟 컨텐츠 개수/용량 및 처리시간(단위: ms)이 명시된다. 
이미 Purge 된 컨텐츠는 다시 Purge되지 않는다. ::

    {
        "version": "2.0.0",
        "method": "purge",
        "status": "OK",
        "result": { "Count": 24, "Size": 3747491, "Time": 12 }
    }
    
``<Purge2Expire>`` 를 통해 특정조건의 Purge를 Expire로 동작하도록 설정할 수 있다.
결과없는 응답에 대해서는 ``<ResCodeNoCtrlTarget>`` 로 HTTP 응답코드를 설정할 수 있다.

.. note::
   
   원본서버가 장애로 인해 모두 배제되었다면 컨텐츠를 갱신할 수 없기 때문에 Purge가 동작하지 않는다.
   

.. _api-cmd-expire:
   
Expire
====================================

타겟 컨텐츠의 TTL을 즉시 만료시킨다. 
Expire후 최초 접근 시점에 원본서버로부터 변경여부를 확인한다. 
변경되지 않았다면 TTL연장만 있을 뿐 컨텐츠 다운로드는 발생하지 않는다. ::

    http://127.0.0.1:10040/command/purge?url=...
    
그 외의 모든 동작은 `Purge`_ 와 동일하다.


.. _api-cmd-expireafter:
   
ExpireAfter
====================================

