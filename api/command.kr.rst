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
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image1.jpg|/css/style.css|/script.js
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

타겟 컨텐츠의 TTL만료 시간을 현재(API호출시점)로부터 입력된 시간(초)만큼 뒤에 설정한다. 
ExpireAfter로 만료시간을 앞당겨 컨텐츠를 더 빨리 갱신하거나, 
반대로 만료시간을 늘려 원본서버 부하를 줄일 수 있다. ::

    http://127.0.0.1:10040/command/expireafter?sec=86400&url=...

함수 호출규격은 `Purge`_ / `Expire`_ 와 유사하지만 sec파라미터(단위: 초)를 통해 
TTL만료 시간을 지정할 수 있다. 
sec가 생략된다면 기본 값은 1일(86400초)로 설정되며 0을 입력할 경우 실패한다. 
결과는 `Purge`_ / `Expire`_ 와 동일하지만 원본서버 장애여부와 상관없이 동작한다. 
결과없는 응답에 대해서는 `<ResCodeNoCtrlTarget>` 로 HTTP 응답코드를 설정할 수 있다.

.. note::
   ExpireAfter는 캐싱되어있는 컨텐츠의 현재 만료시간만을 설정할 뿐 커스텀TTL이나 
   설정된 기본 TTL을 변경시키는 API가 아니다. 
   ExpireAfter 호출뒤에 캐싱된 컨텐츠들은 영향을 받지 않는다.
   
   
   url파라미터를 먼저 입력하는 경우 sec파라미터가 url파라미터의 QueryString으로 인식될 수 있다. 
   그러므로 sec파라미터가 먼저 입력되는 것이 안전하다.
   
   

.. _api-cmd-hardpurge:
   
HardPurge
====================================

`Purge`_ / `Expire`_ / `ExpireAfter`_ 이상의 API는 원본서버 장애상황에서도 컨텐츠가 
사라지지 않고 정상적으로 동작한다. 
하지만 HardPurge는 컨텐츠의 완전한 삭제를 의미한다. 
HardPurge는 가장 강력한 삭제방법이지만 삭제한 컨텐츠는 원본서버에 장애가 발생해도 되살릴 수 없다. 
결과없는 응답에 대해서는 `<ResCodeNoCtrlTarget>` 로 HTTP 응답코드를 설정할 수 있다. ::

    http://127.0.0.1:10040/command/hardpurge?url=...
    

.. _api-cmd-restart:
   
STON 재시작
====================================

STON을 재시작한다. 
이 명령어는 자칫 의도하지 않은 결과를 초래할 수 있다. 
때문에 웹 페이지를 통한 확인작업이 반드시 필요하도록 개발되었다. ::

    http://127.0.0.1:10040/command/restart
    http://127.0.0.1:10040/command/restart?key=JUSTDOIT     // 즉시 실행
    
    
.. _api-cmd-terminate:
   
STON 종료
====================================

STON을 종료한다. 
이 명령어는 자칫 의도하지 않은 결과를 초래할 수 있다. 
때문에 웹 페이지를 통한 확인작업이 반드시 필요하도록 개발되었다. ::

    http://127.0.0.1:10040/command/terminate
    http://127.0.0.1:10040/command/terminate?key=JUSTDOIT     // 즉시 실행
    
    
.. _api-cmd-cacheclear:
   
STON 초기화
====================================

서비스를 중단하고 Caching된 모든 컨텐츠를 삭제한다. 
전역설정(server.xml)에 설정된 모든 디스크를 포맷하며 작업이 완료되면 다시 서비스는 재개된다. ::

    http://127.0.0.1:10040/command/cacheclear
    http://127.0.0.1:10040/command/cacheclear?key=JUSTDOIT     // 즉시 실행
    
콘솔에서는 다음 명령어를 사용하여 전체 또는 하나의 가상호스트를 초기화할 수 있다. ::

    ./stonapi reset
    ./stonapi reset/ston.winesoft.co.kr
    


.. _api-cmd-cleanup:
   
Clean-Up
====================================

Clean-Up을 수행한다. Age파라미터의 단위는 "일"이다. ::

    http://127.0.0.1:10040/command/cleanup
    http://127.0.0.1:10040/command/cleanup?age=10
    
Age파라미터가 0이라면 디스크 공간이 부족하다고 판단될 때만 Clean-Up을 수행한다.
Age파라미터가 주어지면 해당 "일"동안 한번도 접근되지 않은 컨텐츠를 삭제목록에 추가하고 
Clean-up을 수행한다.



.. _api-cmd-diskhotswap:
   
디스크 Hot-Swap
====================================

서비스 중단없이 디스크를 교체한다. 
디스크 파라미터는 `<Storage>` 에 설정된 `<Disk>` 값을 입력한다. ::

    http://127.0.0.1:10040/command/unmount?disk=...
    http://127.0.0.1:10040/command/umount?disk=...
    
배제된 디스크는 즉시 사용되지 않으며 해당 디스크에 저장된 모든 컨텐츠는 무효화된다. 
관리자에 의해 배제된 디스크의 상태는 ``Unmounted`` 로 설정된다. ::

    http://127.0.0.1:10040/command/mount?disk=...
    
디스크를 서비스에 재투입한다. 
재투입된 디스크의 모든 컨텐츠는 무효화된다.



.. _api-cmd-vhostlist:
   
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
   

