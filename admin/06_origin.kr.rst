.. _origin:

원본서버
******************

STON과 원본서버의 관계를 설정한다.
원본서버는 HTTP규격을 준수하는 고객의 서버를 의미한다.

.. toctree::
   :maxdepth: 2

원본보호
====================================

STON의 목표 중 하나는 원본서버를 보호하는 것이다.


장애감지와 자동복구
---------------------

원본서버에 장애가 발생하면 자동배제한다.
다시 안정화되면 서비스에 투입한다. ::

    <OriginOptions>
        <ConnectTimeout>3</ConnectTimeout>
        <ReceiveTimeout>10</ReceiveTimeout>
        <Exclusion>3</Exclusion>
        <Recovery Cycle="10">5</Recovery>
    </OriginOptions>

-  ``<ConnectTimeout> (기본: 3초)``

    n초 이내에 원본서버와 접속이 이루어지지 않는 경우 접속실패로 간주한다.
   
-  ``<ReceiveTimeout> (기본: 10초)``

    정상적인 HTTP요청에도 불구하고 원본서버가 HTTP응답/컨텐츠를 n초 동안 
    보내지 않는 경우 전송실패로 간주한다.   

-  ``<Exclusion> (기본: 3회)``

    원본서버에서 연속적으로 n번 장애상황(연결실패 또는 전송실패)이 발생하면 
    해당 서버를 유효 원본서버 목록에서 배제한다. 
    배제 전 정상적인 연결이 이루어진다면 이 값은 다시 0으로 초기화된다.

-  ``<Recovery> (기본: 5회, Cycle: 10초)``

    배제된 원본서버를 다시 서비스에 투입하기 위하여 설정된 ``Cycle`` 마다 "/"를 요청한다. 
    원본서버로 연결이 되고 (응답코드 여부와 상관없이)응답이 n회 되면 
    서비스에 재투입한다. 


DNS
---------------------

원본서버 주소가 Domain으로 설정되어 있다면 항상 최신의 Resolving결과가 사용된다. ::
하지만 DNS서버의 장애 또는 네트워크 구간 장애로 인하여 Resolving이 실패할 경우 
마지막으로 Resolving된 IP에 부하가 집중될 수 있다. 

    <OriginOptions>
        <DNSBackup>5min</DNSBackup>
    </OriginOptions>

이런 문제를 극복하기 위하여 Resolving장애 상황이 발생하면 최근(일정시간 동안) 
``DNSBackup`` 시간동안 사용된 Unique한 IP들을 모두 사용하여 부하를 분산한다. ::
