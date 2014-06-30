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
   


실시간/5분 통계
====================================

STON의 모든 통계는 가상호스트별로 따로 수집될 뿐만 아니라 실시간, 5분 평균으로 제공된다. 
고객이 통계를 보다 쉽게 분석, 가공할 수 있도록 JSON과 XML 포맷으로 제공한다. 


호스트(종합) 통계
---------------------

Host통계는 가장 상위 개념의 통계로 서비스하는 모든 가상호스트의 통계를 종합하여 제공한다. 

다음은 JSON 통계이다. ::

   {
     "Host":
     {
       "Version":"0.9.6.2",
       "Name":"localhost",
       "State":"Healthy",
       "Uptime":155996,
       "OriginSession":33,
       "OriginActiveSession":20,
       "OriginInbound":688177,
       "OriginOutbound":14184,
       "OriginReqCount":62,
       "OriginResTotalCount":62,
       "OriginResTotalTimeRes":2375,
       "OriginResTotalTimeComplete":2509,
       "OriginRes2xxCount":54,
       "OriginRes2xxTimeRes":2327,
       "OriginRes2xxTimeComplete":2481,
       "OriginRes3xxCount":8,
       "OriginRes3xxTimeRes":2700,
       "OriginRes3xxTimeComplete":2700,
       "OriginRes4xxCount":0,
       "OriginRes4xxTimeRes":0,
       "OriginRes4xxTimeComplete":0,
       "OriginRes5xxCount":0,
       "OriginRes5xxTimeRes":0,
       "OriginRes5xxTimeComplete":0,
       "ClientSession":155,
       "ClientActiveSession":80
       "ClientInbound":35748,
       "ClientOutbound":972906,
       "ClientReqCount":152,
       "ClientResTotalCount":152,
       "ClientResTotalTimeRes":1411,
       "ClientResTotalTimeComplete":1479,
       "ClientRes2xxCount":93,
       "ClientRes2xxTimeRes":2305,
       "ClientRes2xxTimeComplete":2409,
       "ClientRes3xxCount":59,
       "ClientRes3xxTimeRes":3,
       "ClientRes3xxTimeComplete":13,
       "ClientRes4xxCount":0,
       "ClientRes4xxTimeRes":0,
       "ClientRes4xxTimeComplete":0,
       "ClientRes5xxCount":0,
       "ClientRes5xxTimeRes":0,
       "ClientRes5xxTimeComplete":0,
       "RequestHitRatio":6387,
       "ByteHitRatio":2926,
       "HttpCountSum" : 
       {
         "OriginReqCount" : 0, 
         "OriginResTotalCount" : 0, 
         "OriginRes2xxCount" : 0, 
         "OriginRes3xxCount" : 0, 
         "OriginRes4xxCount" : 0, 
         "OriginRes5xxCount" : 0, 
         "ClientReqCount" : 0, 
         "ClientResTotalCount" : 0, 
         "ClientRes2xxCount" : 0, 
         "ClientRes3xxCount" : 0, 
         "ClientRes4xxCount" : 0, 
         "ClientRes5xxCount" : 0 
       },
       "HttpRequestHitSum" : 
       {
         "TCP_NONE" : 0, 
         "TCP_HIT" : 0, 
         "TCP_IMS_HIT" : 0, 
         "TCP_REFRESH_HIT" : 0, 
         "TCP_REF_FAIL_HIT" : 0, 
         "TCP_NEGATIVE_HIT" : 0, 
         "TCP_REDIRECT_HIT" : 0, 
         "TCP_MISS" : 0, 
         "TCP_REFRESH_MISS" : 0, 
         "TCP_CLIENT_REFRESH_MISS" : 0, 
         "TCP_DENIED" : 0, 
         "TCP_ERROR" : 0
       },
       "FileSystem":
       {
         "RequestHitRatio":0,
         "ByteHitRatio":0,
         "Outbound":0,
         "Session":0
       },
       "System":{ ... },
       "VirtualHost": [ ... ]
       "View": [ ... ]
     }
   }
   
다음은 XML 통계이다. ::

   <Host 
     Version="0.9.6.2"
     Name="localhost"
     State="Healthy"
     Uptime="155986"
     OriginSession="32"
     OriginActiveSession="20" 
     OriginInbound="1140741"
     OriginOutbound="10059"
     OriginReqCount="42"
     OriginResTotalCount="42"
     OriginResTotalTimeRes="5071"
     OriginResTotalTimeComplete="10288"
     OriginRes2xxCount="19"
     OriginRes2xxTimeRes="9989"
     OriginRes2xxTimeComplete="21521"
     OriginRes3xxCount="23"
     OriginRes3xxTimeRes="1008"
     OriginRes3xxTimeComplete="1008"
     OriginRes4xxCount="0"
     OriginRes4xxTimeRes="0"
     OriginRes4xxTimeComplete="0"
     OriginRes5xxCount="0"
     OriginRes5xxTimeRes="0"
     OriginRes5xxTimeComplete="0"
     ClientSession="165"
     ClientActiveSession="80"
     ClientInbound="14792"
     ClientOutbound="1981700"
     ClientReqCount="64"
     ClientResTotalCount="64"
     ClientResTotalTimeRes="5535"
     ClientResTotalTimeComplete="6840"
     ClientRes2xxCount="44"
     ClientRes2xxTimeRes="8050"
     ClientRes2xxTimeComplete="9943"
     ClientRes3xxCount="20"
     ClientRes3xxTimeRes="5"
     ClientRes3xxTimeComplete="15"
     ClientRes4xxCount="0"
     ClientRes4xxTimeRes="0"
     ClientRes4xxTimeComplete="0"
     ClientRes5xxCount="0"
     ClientRes5xxTimeRes="0"
     ClientRes5xxTimeComplete="0"
     RequestHitRatio="6923"
     ByteHitRatio="4243">
     <HttpCountSum 
       OriginReqCount="0" 
       OriginResTotalCount="0" 
       OriginRes2xxCount="0" 
       OriginRes3xxCount="0" 
       OriginRes4xxCount="0" 
       OriginRes5xxCount="0" 
       ClientReqCount="0" 
       ClientResTotalCount="0" 
       ClientRes2xxCount="0" 
       ClientRes3xxCount="0" 
       ClientRes4xxCount="0" 
       ClientRes5xxCount="0"/>
     <HttpRequestHitSum 
       TCP_NONE="0" 
       TCP_HIT="0" 
       TCP_IMS_HIT="0" 
       TCP_REFRESH_HIT="0" 
       TCP_REF_FAIL_HIT="0" 
       TCP_NEGATIVE_HIT="0" 
       TCP_REDIRECT_HIT="0" 
       TCP_MISS="0" 
       TCP_REFRESH_MISS="0" 
       TCP_CLIENT_REFRESH_MISS="0" 
       TCP_DENIED="0" 
       TCP_ERROR="0"/>
     <FileSystem>
       <RequestHitRatio>0</RequestHitRatio>
       <ByteHitRatio>0</ByteHitRatio>
       <Outbound>0</Outbound>
       <Session>0</Session>
     </FileSystem>
     <System> ... </System>
     <VirtualHost> ... </VirtualHost>
     <VirtualHost> ... </VirtualHost>
     <VirtualHost> ... </VirtualHost>
     <View> ... </View>
     <View> ... </View>
   </Host>
   
aaa