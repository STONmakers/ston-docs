.. _api-cmd:

/Command
******************

/command 계열 API는 STON을 제어하는 역할을 한다.

.. toctree::
   :maxdepth: 2






    

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

