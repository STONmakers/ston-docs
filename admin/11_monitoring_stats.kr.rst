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

호스트 통계는 가장 상위 개념의 통계로 서비스하는 모든 가상호스트의 통계를 종합한다. 
같은 데이터를 JSON과 XML형식으로 제공한다. ::

   {                                            <Host                                    
     "Host":                                      Version="2.0.0"                       
     {                                            Name="localhost"                       
       "Version":"2.0.0",                         State="Healthy"                        
       "Name":"localhost",                        Uptime="155986"                        
       "State":"Healthy",                         OriginSession="32"                     
       "Uptime":155996,                           OriginActiveSession="20"               
       "OriginSession":33,                        OriginInbound="1140741"                
       "OriginActiveSession":20,                  OriginOutbound="10059"                 
       "OriginInbound":688177,                    OriginReqCount="42"                    
       "OriginOutbound":14184,                    OriginResTotalCount="42"               
       "OriginReqCount":62,                       OriginResTotalTimeRes="5071"           
       "OriginResTotalCount":62,                  OriginResTotalTimeComplete="10288"     
       "OriginResTotalTimeRes":2375,              OriginRes2xxCount="19"                 
       "OriginResTotalTimeComplete":2509,         OriginRes2xxTimeRes="9989"             
       "OriginRes2xxCount":54,                    OriginRes2xxTimeComplete="21521"       
       "OriginRes2xxTimeRes":2327,                OriginRes3xxCount="23"                 
       "OriginRes2xxTimeComplete":2481,           OriginRes3xxTimeRes="1008"             
       "OriginRes3xxCount":8,                     OriginRes3xxTimeComplete="1008"        
       "OriginRes3xxTimeRes":2700,                OriginRes4xxCount="0"                  
       "OriginRes3xxTimeComplete":2700,           OriginRes4xxTimeRes="0"                
       "OriginRes4xxCount":0,                     OriginRes4xxTimeComplete="0"           
       "OriginRes4xxTimeRes":0,                   OriginRes5xxCount="0"                  
       "OriginRes4xxTimeComplete":0,              OriginRes5xxTimeRes="0"                
       "OriginRes5xxCount":0,                     OriginRes5xxTimeComplete="0"           
       "OriginRes5xxTimeRes":0,                   ClientSession="165"                    
       "OriginRes5xxTimeComplete":0,              ClientActiveSession="80"               
       "ClientSession":155,                       ClientInbound="14792"                  
       "ClientActiveSession":80                   ClientOutbound="1981700"               
       "ClientInbound":35748,                     ClientReqCount="64"                    
       "ClientOutbound":972906,                   ClientResTotalCount="64"               
       "ClientReqCount":152,                      ClientResTotalTimeRes="5535"           
       "ClientResTotalCount":152,                 ClientResTotalTimeComplete="6840"      
       "ClientResTotalTimeRes":1411,              ClientRes2xxCount="44"                 
       "ClientResTotalTimeComplete":1479,         ClientRes2xxTimeRes="8050"             
       "ClientRes2xxCount":93,                    ClientRes2xxTimeComplete="9943"        
       "ClientRes2xxTimeRes":2305,                ClientRes3xxCount="20"                 
       "ClientRes2xxTimeComplete":2409,           ClientRes3xxTimeRes="5"                
       "ClientRes3xxCount":59,                    ClientRes3xxTimeComplete="15"          
       "ClientRes3xxTimeRes":3,                   ClientRes4xxCount="0"                  
       "ClientRes3xxTimeComplete":13,             ClientRes4xxTimeRes="0"                
       "ClientRes4xxCount":0,                     ClientRes4xxTimeComplete="0"           
       "ClientRes4xxTimeRes":0,                   ClientRes5xxCount="0"                  
       "ClientRes4xxTimeComplete":0,              ClientRes5xxTimeRes="0"                
       "ClientRes5xxCount":0,                     ClientRes5xxTimeComplete="0"           
       "ClientRes5xxTimeRes":0,                   RequestHitRatio="6923"                 
       "ClientRes5xxTimeComplete":0,              ByteHitRatio="4243">                   
       "RequestHitRatio":6387,                    <HttpCountSum                          
       "ByteHitRatio":2926,                         OriginReqCount="0"                   
       "HttpCountSum" :                             OriginResTotalCount="0"              
       {                                            OriginRes2xxCount="0"                
         "OriginReqCount" : 0,                      OriginRes3xxCount="0"                
         "OriginResTotalCount" : 0,                 OriginRes4xxCount="0"                
         "OriginRes2xxCount" : 0,                   OriginRes5xxCount="0"                
         "OriginRes3xxCount" : 0,                   ClientReqCount="0"                   
         "OriginRes4xxCount" : 0,                   ClientResTotalCount="0"              
         "OriginRes5xxCount" : 0,                   ClientRes2xxCount="0"                
         "ClientReqCount" : 0,                      ClientRes3xxCount="0"                
         "ClientResTotalCount" : 0,                 ClientRes4xxCount="0"                
         "ClientRes2xxCount" : 0,                   ClientRes5xxCount="0"/>              
         "ClientRes3xxCount" : 0,                 <HttpRequestHitSum                     
         "ClientRes4xxCount" : 0,                   TCP_NONE="0"                         
         "ClientRes5xxCount" : 0                    TCP_HIT="0"                          
       },                                           TCP_IMS_HIT="0"                      
       "HttpRequestHitSum" :                        TCP_REFRESH_HIT="0"                  
       {                                            TCP_REF_FAIL_HIT="0"                 
         "TCP_NONE" : 0,                            TCP_NEGATIVE_HIT="0"                 
         "TCP_HIT" : 0,                             TCP_REDIRECT_HIT="0"                 
         "TCP_IMS_HIT" : 0,                         TCP_MISS="0"                         
         "TCP_REFRESH_HIT" : 0,                     TCP_REFRESH_MISS="0"                 
         "TCP_REF_FAIL_HIT" : 0,                    TCP_CLIENT_REFRESH_MISS="0"          
         "TCP_NEGATIVE_HIT" : 0,                    TCP_DENIED="0"                       
         "TCP_REDIRECT_HIT" : 0,                    TCP_ERROR="0"/>                      
         "TCP_MISS" : 0,                          <FileSystem>                           
         "TCP_REFRESH_MISS" : 0,                    <RequestHitRatio>0</RequestHitRatio> 
         "TCP_CLIENT_REFRESH_MISS" : 0,             <ByteHitRatio>0</ByteHitRatio>       
         "TCP_DENIED" : 0,                          <Outbound>0</Outbound>               
         "TCP_ERROR" : 0                            <Session>0</Session>                 
       },                                         </FileSystem>                          
       "FileSystem":                              <System> ... </System>                 
       {                                          <VirtualHost> ... </VirtualHost>       
         "RequestHitRatio":0,                     <VirtualHost> ... </VirtualHost>       
         "ByteHitRatio":0,                        <VirtualHost> ... </VirtualHost>       
         "Outbound":0,                            <View> ... </View>                     
         "Session":0                              <View> ... </View>                     
       },                                       </Host>                                  
       "System":{ ... },                                                                 
       "VirtualHost": [ ... ]
       "View": [ ... ]
     }
   }
   
-  ``Version`` STON 버전
-  ``Name`` 호스트이름. 설정하지 않았다면 시스템 이름을 보여준다.
-  ``State`` 서비스 상태. (Healthy=정상 서비스, Inactive=라이센스 비활성화, Emergency)
-  ``Uptime (단위: 초)`` 서비스 실행시간
-  ``OriginSession`` 원본세션 수
-  ``OriginActiveSession`` 전송 중인 원본세션 수
-  ``OriginInbound (단위: Bytes, 평균)`` 원본서버부터 받은 양
-  ``OriginReqCount (평균)`` 원본서버로 보낸 요청횟수
-  ``OriginOutbound (단위: Bytes, 평균)`` 원본서버로 보낸 양
-  ``OriginResTotalCount (평균)`` 원본서버 응답횟수
-  ``OriginResTotalTimeRes (단위: 0.01ms, 평균)`` 원본서버 응답시간 (HTTP요청 전송 ~ HTTP응답 첫 수신)
-  ``OriginResTotalTimeComplete (단위: 0.01ms, 평균)`` 원본서버 HTTP 트랜잭션 완료시간 (HTTP요청 전송 ~ HTTP응답 완료)
-  ``OriginRes2xxCount (평균)`` 원본서버 2xx응답횟수
-  ``OriginRes2xxTimeRes (단위: 0.01ms, 평균)`` 원본서버 2xx응답시간
-  ``OriginRes2xxTimeComplete (단위: 0.01ms, 평균)`` 원본서버 2xx 트랜잭션 완료시간
-  ``OriginRes3xxCount (평균)`` 원본서버 3xx응답횟수
-  ``OriginRes3xxTimeRes (단위: 0.01ms, 평균)`` 원본서버 3xx응답시간
-  ``OriginRes3xxTimeComplete (단위: 0.01ms, 평균)`` 원본서버 3xx 트랜잭션 완료시간
-  ``OriginRes4xxCount (평균)`` 원본서버 4xx응답횟수
-  ``OriginRes4xxTimeRes (단위: 0.01ms, 평균)`` 원본서버 4xx응답시간
-  ``OriginRes4xxTimeComplete (단위: 0.01ms, 평균)`` 원본서버 4xx 트랜잭션 완료시간
-  ``OriginRes5xxCount (평균)`` 원본서버 5xx응답횟수
-  ``OriginRes5xxTimeRes (단위: 0.01ms, 평균)`` 원본서버 5xx응답시간
-  ``OriginRes5xxTimeComplete (단위: 0.01ms, 평균)`` 원본서버 5xx 트랜잭션 완료시간
-  ``ClientSession`` 클라이언트 세션 수
-  ``ClientActiveSession`` 전송 중인 클라이언트 세션 수
-  ``ClientInbound (단위: Bytes, 평균)`` 클라이언트로부터 받은 양
-  ``ClientOutbound (단위: Bytes, 평균)`` 클라이언트로에게 보낸 양
-  ``ClientReqCount (평균)`` 클라이언트로 받은 요청횟수
-  ``ClientResTotalCount (평균)`` 클라이언트 응답횟수
-  ``ClientResTotalTimeRes (단위: 0.01ms, 평균)`` 클라이언트 응답시간 (HTTP요청 수신 ~ HTTP응답 전송)
-  ``ClientResTotalTimeComplete (단위: 0.01ms, 평균)`` 클라이언트 HTTP 트랜잭션 완료시간 (HTTP요청 수신 ~ HTTP응답 완료)
-  ``ClientRes2xxCount (평균)`` 클라이언트 2xx응답횟수
-  ``ClientRes2xxTimeRes (단위: 0.01ms, 평균)`` 클라이언트 2xx응답시간
-  ``ClientRes2xxTimeComplete (단위: 0.01ms, 평균)`` 클라이언트 2xx 트랜잭션 완료시간
-  ``ClientRes3xxCount (평균)`` 클라이언트 3xx응답횟수
-  ``ClientRes3xxTimeRes (단위: 0.01ms, 평균)`` 클라이언트 3xx응답시간
-  ``ClientRes3xxTimeComplete (단위: 0.01ms, 평균)`` 클라이언트 3xx 트랜잭션 완료시간
-  ``ClientRes4xxCount (평균)`` 클라이언트 4xx응답횟수
-  ``ClientRes4xxTimeRes (단위: 0.01ms, 평균)`` 클라이언트 4xx응답시간
-  ``ClientRes4xxTimeComplete (단위: 0.01ms, 평균)`` 클라이언트 4xx 트랜잭션 완료시간
-  ``ClientRes5xxCount (평균)`` 클라이언트 5xx응답횟수
-  ``ClientRes5xxTimeRes (단위: 0.01ms, 평균)`` 클라이언트 5xx응답시간
-  ``ClientRes5xxTimeComplete (단위: 0.01ms, 평균)`` 클라이언트 5xx 트랜잭션 완료시간
-  ``RequestHitRatio (단위: 0.01%, 평균)`` Hit율. 캐싱객체가 생성되어 있고 해당 객체가 초기화되어 있다면 Hit이다. 반대로 캐싱객체가 없거나 해당 객체가 원본서버로부터 초기화되지 않았다면 Hit로 치지 않는다. 응답코드와 Hit율은 관련이 없다.
-  ``ByteHitRatio (단위: 0.01%, 평균)`` 원본서버 대비 클라이언트 전송률. ::

      (클라이언트 Outbound - 원본서버 Inbound) / 클라이언트 Outbound
      
   원본서버가 훨씬 빠른 속도를 가지고 있거나 클라이언트 세션이 금방 끊어진다면 음수가 된다.
   
-  ``FileSystem`` 독립적인 FileSystem 통계로 다른 통계 수치에 취합되지 않는다.

   - ``RequestHitRatio (단위: 0.01%, 평균)`` File I/O를 통한 Hit율
   - ``ByteHitRatio (단위: 0.01%, 평균)`` 원본서버 대비 File I/O 전송률
   - ``Outbound (단위: Bytes, 평균)`` File I/O로 서비스한 데이터 크기
   - ``Session (평균)`` File I/O 진행 중인 Thread 수

-  **HitRatio의 이해**
   
   .. figure:: img/stat_filesystem1.png
      :align: center
      
      HTTP와 File I/O는 가상호스트를 공유한다.
      
   Apache를 통해 접근되는 File I/O의 RequestHitRatio는 0%이 된다.
   하지만 HTTP Server의 경우 File I/O에 의해 캐싱된 파일을 접근하기 때문에 100%의 RequestHitRatio를 가진다. 
   ByteHitRatio의 경우 원본 Inbound대비 Http outbound, File I/O outbound로 각각 계산된다.
   
.. note::

   5분 통계에서만 제공되는 항목.
   
   -  ``HttpCountSum`` 5분간 발생한 HTTP 트랜잭션의 총 개수
   
   -  ``HttpRequestHitSum`` 5분간 발생한 캐시 HIT 결과