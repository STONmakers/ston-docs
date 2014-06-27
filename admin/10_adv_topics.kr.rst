.. _adv_topics:

고급기능
******************

다루지 않았던 기능들에 대해 설명한다.

.. toctree::
   :maxdepth: 2

Emergency 모드
====================================

STON은 모든 가상호스트가 MemoryBlock을 공유하면서 데이터를 관리하도록 설계되어 있다. 
신규 메모리가 필요한 경우 참조되지 않는 오래된 MemoryBlock을 재사용하여 신규 메모리를 확보한다. 
이 과정을 Memory-Swap이라고 부른다. 
이런 구조를 통해 STON을 장기간 운영하여도 안정성을 확보할 수 있다.

.. figure:: img/faq_emergency1.png
   :align: center
      
   콘텐츠 데이터는 MemoryBlock에 담겨 서비스된다.

위 그림의 우측 상황처럼 모든 MemoryBlock이 사용 중이어서 재사용할 수 있는 MemoryBlock이 
존재하지 않는 상황이 발생할 수 있다. 
이때는 Memory-Swap이 불가능해진다. 
예를 들어 모든 클라이언트가 서로 다른 데이터 영역을 아주 조금씩 다운로드 받거나 
원본서버에서 서로 다른 데이터를 아주 조금씩 전송하는 상황이 동시에 발생하는 경우가 최악이다. 
이런 경우 시스템으로부터 새로운 메모리를 할당받아 STON이 사용하는 것도 방법이다. 
하지만 이런 상황이 지속될 경우 STON의 메모리 사용량이 높아진다. 
메모리 사용량이 과도하게 높아질 경우 시스템 메모리스왑을 발생시키거나 최악의 경우 
OS가 STON을 종료시키는 상황이 발생할 수 있다.

.. note::

   Emergency 모드란 메모리 부족상황이 발생할 경우 임시적으로 신규 MemoryBlock의 할당을 금지시키는 상황을 의미한다.

이는 과다 메모리 사용으로부터 STON을 보호하기 위한 방법이며, 
재사용가능한 MemoryBlock이 충분히 확보되면 자동 해지된다. ::

    <Cache>
        <EmergencyMode>OFF</EmergencyMode>
    </Cache>
    
-  ``<EmergencyMode>``

   - ``OFF (기본)`` 사용하지 않는다.
   
   - ``ON`` 사용한다.

Emergency모드일 때 STON은 다음과 같이 동작합니다.

- 이미 로딩되어 있는 컨텐츠는 정상적으로 서비스된다.
- 바이패스는 정상적으로 이루어진다.
- 로딩되어 있지 않은 컨텐츠에 대해서는 503 service temporarily unavailable로 응답한다. TCP_ERROR상태가 증가한다.
- Idle 클라이언트 소켓을 빠르게 정리한다.
- 신규 컨텐츠를 캐싱할 수 없다.
- TTL이 만료된 컨텐츠를 갱신하지 않는다.
- SNMP의 cache.vhost.status와 XML/JSON통계의 Host.State 값이 "Emergency"로 제공된다.
- Info로그에 Emergency모드로 전환/해제를 다음과 같이 기록한다. ::

    2013-08-07 21:10:42 [WARNING] Emergency mode activated. (Memory overused: +100.23MB)
    ...(생략)...
    2013-08-07 21:10:43 [NOTICE] Emergency mode inactivated.


    