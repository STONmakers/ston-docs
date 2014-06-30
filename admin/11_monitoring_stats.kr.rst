.. _monitoring_stats:

모니터링 & 통계
******************

1초 통계에 기반하여 실시간 모니터링 및 서비스 통계가 수집된다.


.. toctree::
   :maxdepth: 2

통계수집
====================================

관련설정을 통해 통계수집 범위를 조절할 수 있다. ::

    <Stats>
        <DirDepth>0</DirDepth>
        <DirDepthAccum>OFF</DirDepthAccum>
        <HttpsTraffic>OFF</HttpsTraffic>
        <ClientLocal>OFF</ClientLocal>
        <OriginLocal>OFF</OriginLocal>
    </Stats>
    
-  ``<DirDepth> (기본: 0)`` 
   
   디렉토리별로 통계를 수집한다. 
   0으로 설정된 경우 모든 통계를 루트(/) 디렉토리로 수집한다. 
   1로 설정하면 통계는 첫 번째 Depth 디렉토리별로 수집된다. 
   
   .. note:
      
      값의 제한은 없지만 수 만개 이상의 디렉토리 통계를 수집할 경우 자칫 메모리 문제를 초래할 수 있다.
   
-  ``<DirDepthAccum>`` 

   디렉토리별로 통계수집할 때 상위 디렉토리 통계 합산여부를 설정한다.
   ``<DirDepth>`` 이 0이라면 이 설정은 무시된다.
   
   - ``OFF (기본)`` 상위 디렉토리로 통계를 합산하지 않는다.
   
   - ``ON`` 상위 디렉토리로 통계를 합산한다.
   
   예를 들어, ``<DirDepth>`` 이 2이고 모든 디렉토리에 동일하게 10만큼의 트래픽이 
   발생하고 있다고 가정합니다. 
   ``<DirDepthAccum>`` 이 ``OFF`` 라면 좌측 그림처럼 트래픽이 발생하는 디렉토리별로 
   따로 통계가 수집되지만, ``ON`` 이라면 우측 그림처럼 하위 디렉토리의 모든 통계가 
   부모 디렉토리로 누적된다.
   
   .. figure:: img/stats_dirdepth.jpg
      :align: center
          
      상위 디렉토리 누적통계
   
   예를 들어 /img 디렉토리는 하위 디렉토리의 트래픽과 자신의 트래픽을 더한 30을 
   통계 값으로 가지며 이 트래픽은 부모 디렉토리로 합산된다.

-  ``<HttpsTraffic>`` 

   - ``OFF (기본)`` HTTPS트래픽을 SSL통계로만 수집한다.
   
   - ``ON`` HTTPS트래픽을 SSL과 HTTP양쪽 통계에 같이 수집한다. 
   
   기본적으로 SSL레이어를 통과하면 별도의 SSL 통계로 수집한다.
   HTTPS의 경우 상위 프로토콜에서 HTTP로 처리되기 때문에 보다 세세한 통계수집이 가능하다. 
   하지만 SSL통계와 HTTP통계 양쪽에 중복 통계수집이 되므로 HTTP통계만을 신뢰할 것을 권장한다.

-  ``<ClientLocal>`` 

   Loopback 클라이언트와 STON구간의 트래픽을 통계로 집계한다.

   - ``OFF (기본)`` 집계하지 않는다.
   
   - ``ON`` 집계한다.

-  ``<OriginLocal>`` 

   STON구간과 Loopback 원본서버 구간의 트래픽을 통계로 집계한다.

   - ``OFF (기본)`` 집계하지 않는다.
   
   - ``ON`` 집계한다.