.. _multimedia:

미디어
******************

클라이언트 환경과 서비스의 다양화와 함께 콘텐츠를 다양한 형태로 가공할 필요가 있다.
같은 콘텐츠지만 다양한 형태로 원본서버에 존재하는 경우가 매우 많으며, 
이는 처리시간과 저장공간의 낭비로 이어질 뿐만 아니라 관리의 어려움을 초래한다.
STON은 단일한 콘텐츠를 다양한 형태로 동적가공하여 서비스한다.

.. toctree::
   :maxdepth: 2

Video/Audio
====================================

MP4, M4A, MP3 등 다양한 Video/Audio 포맷을 지원한다.

헤더위치 변경
-----------------------

보통 MP4포맷의 경우 인코딩 과정 중에는 헤더를 완성할 수 없기 때문에 
완료 후 파일의 맨 뒤에 붙인다. (헤더를 앞으로 옮기려면 별도의 처리가 필요하다.)
헤더가 뒤에 있다면 이를 지원하지 않는 플레이어에서 Pseudo-Streaming이 불가능하다. 
헤더 위치변경은 전송단계에서만 발생할 뿐 원본의 형태를 변경하지 않는다.
별도의 저장공간을 사용하지도 않는다. ::

    <Media>
        <UpfrontMP4Header>OFF</UpfrontMP4Header>
        <UpfrontM4AHeader>OFF</UpfrontM4AHeader>
    </Media>

-  ``<UpfrontMP4Header>``
   
   - ``OFF (기본)`` 아무 것도 하지 않는다.
   
   - ``ON`` 확장자가 .mp4이고 헤더가 뒤에 있다면 헤더를 앞으로 옮겨서 전송한다.

-  ``<UpfrontM4AHeader>``

   - ``OFF (기본)`` 아무 것도 하지 않는다.
   
   - ``ON`` 확장자가 .m4a이고 헤더가 뒤에 있다면 헤더를 앞으로 옮겨서 전송한다.

처음 요청되는 콘텐츠의 헤더를 앞으로 옮겨야 한다면 헤더를 옮기기위해 필요한 
부분을 우선적으로 다운로드 받는다.
아주 스마트할 뿐만 아니라 매우 빠르게 동작한다.
커튼 뒤의 복잡한 과정과는 상관없이, 클라이언트는 원래부터 헤더가 앞에 있는 
온전한 파일을 서비스 받는다.

.. note::

   분석할 수 없거나 깨진 파일이라면 원본형태 그대로 서비스된다.


Trimming
-----------------------

시간 값을 기준으로 원하는 구간을 추출한다.
Trimming은 전송단계에서만 발생할 뿐 원본의 형태를 변경하지 않는다.
별도의 저장공간을 사용하지도 않는다. 
파라미터는 클라이언트 QueryString을 통해 입력받는다. ::

    <Media>
        <MP4Trimming StartParam="start" EndParam="end">OFF</MP4Trimming>
        <MP3Trimming StartParam="start" EndParam="end">OFF</MP3Trimming>
        <M4ATrimming StartParam="start" EndParam="end">OFF</M4ATrimming>
    </Media>

-  ``<MP4Trimming>`` ``<MP3Trimming>`` ``<M4ATrimming>``
   
   - ``OFF (기본)`` 아무 것도 하지 않는다.
   
   - ``ON`` 확장자(.mp4, .mp3, .m4a)가 일치하면 원하는 구간만큼 서비스하도록 Trimming한다.
     Trimming구간은 ``StartParam`` 속성과 ``EndParam`` 으로 설정한다.
     
예를 들어 10분 분량의 동영상(/video.mp4)을 특정 구간만 Trimming하고 싶다면 
QueryString에 원하는 시점(단위: 초)을 명시한다. ::

    http://vod.wineosoft.co.kr/video.mp4                // 10분 : 전체 동영상
    http://vod.wineosoft.co.kr/video.mp4?end=60         // 1분 : 처음부터 60초까지
    http://vod.wineosoft.co.kr/video.mp4?start=120      // 8분 : 2분(120초)부터 끝까지
    http://vod.wineosoft.co.kr/video.mp4?start=3&end=13 // 10초 : 3초부터 13초까지

``StartParam`` 값이 ``EndParam`` 값보다 클 경우 구간이 지정되지 않은 것으로 판단하여 
원본을 서비스한다.

이 기능은 HTTP Pseudo-Streaming으로 구현된 동영상 플레이어의 Skip기능을 위해서 
개발되었습니다. 그러므로 Range요청을 처리하는 것처럼 파일을 Offset기반으로 자르지 않고 
올바르게 재생될 수 있도록 키프레임과 시간을 인지하여 구간을 추출한다. 
클라이언트에게 전달되는 파일은 다음 그림처럼 MP4헤더가 재생성된 온전한 형태의 MP4파일이다.

.. figure:: img/conf_media_mp4trimming.png
   :align: center
      
   완전한 형태의 파일이 제공된다.

추출된 구간은 별도의 파일로 인식되기 때문에 200 OK로 응답된다. 
그러므로 다음과 같이 Range헤더가 명시된 경우 추출된 파일로부터 Range를 계산하여 
"206 Particial Content" 로 응답한다.

.. figure:: img/conf_media_mp4trimming_range.png
   :align: center
      
   평범한 Range요청처럼 처리된다.
   
구간추출 파라미터가 QueryString 표현을 사용하기 때문에 자칫 ``<ApplyQueryString>`` 과 
헷갈릴 수 있다. ``<ApplyQueryString>`` 설정이 ``ON`` 인 경우 클라이언트가 요청한 URL의 
QueryString이 모두 인식되지만 ``StartParam`` 과 ``EndParam`` 은 제외된다. ::

    GET /video.mp4?start=30&end=100
    GET /video.mp4?tag=3277&start=30&end=100&date=20130726
    
예를 들어 위와 같이 ``StartParam`` 이 **start** 로 ``EndParam`` 이 **end** 로 입력된 경우 
이 값들은 구간을 추출하는데 쓰일 뿐 Caching-Key를 생성하거나 원본서버로 요청을 보낼 때는 
생략된다. 각각 다음과 같이 인식된다. ::

    GET /video.mp4
    GET /video.mp4?tag=3277&date=20130726
    
또한 QueryString파라미터는 Module이나 CDN솔루션에 따라 달라질 수 있다. 

.. figure:: img/conf_media_mp4trimming_range.png
   :align: center
   
   JW Player에서 제공하고 있는 Module/CDN별 참고자료
   
이외의 nginx의 `ngx_http_mp4_module <http://nginx.org/en/docs/http/ngx_http_mp4_module.html>`_ 과, 
lighttpd의 `Mod-H264-Streaming-Testing-Version2 <http://h264.code-shop.com/trac/wiki/Mod-H264-Streaming-Testing-Version2>`_ 에서도 
모두 **start** 를 QueryString으로 사용하고 있다.


HTTP Live Streaming
-----------------------

MP4파일을 HLS(HTTP Live Streaming)로 서비스한다. 
원본서버는 더 이상 HLS서비스를 위해 파일을 분할할 필요가 없다. 
MP4파일 헤더의 위치에 상관없이 다운로드와 동시에 실시간으로 .m3u8/.ts파일 변환 후 서비스한다. 
기존 방식의 경우 Pseudo Streaming과 HLS를 위해 다음과 같이 원본파일이 각각 존재해야 한다. 
이런 경우 STON 역시 원본 파일을 그대로 복제하여 고객에게 서비스한다. 
하지만 재생시간이 길수록 파생파일은 많아지며 관리의 어려움은 증가한다.

.. figure:: img/conf_media_mp4hls1.png
   :align: center
   
   수고가 많은 HLS
   
STON의 ``<MP4HLS>`` 는 원본파일로부터 HLS서비스에 필요한 파일을 자동으로 생성한다.

.. figure:: img/conf_media_mp4hls2.png
   :align: center
   
   똑똑한 HLS

모든 .m3u8/.ts파일은 원본파일에서 파생되며 별도의 저장공간을 소비하지 않는다. 
서비스 즉시 메모리에 임시적으로 생성되며 서비스되지 않을 때 자동으로 없어진다. ::

    <Media>    
        <MP4HLS Status="Inactive" Keyword="mp4hls">
            <Index>index.m3u8</Index>
            <Sequence>0</Sequence>
            <Duration>10</Duration>
        </MP4HLS>
    </Media>
    
-  ``<MP4HLS>``
   
   ``Status`` 속성이 ``Active`` 일 때 활성화된다.   


예를 들어 서비스 주소가 다음과 같다면 해당 주소로 Pseudo Streaming을 진행할 수 있다. ::

    http://www.winesoft.com/video.mp4
    
STON은 ``<MP4HLS>`` 에 정의된 ``Keyword`` 문자열을 인식함으로써 HLS서비스를 진행한다. 
다음 URL이 호출되면 /video.mp4로부터 index.m3u8파일을 생성한다. 
인덱스 파일명은 ``<Index>`` 에서 문자열로 설정한다. ::

    http://www.winesoft.com/video.mp4/mp4hls/index.m3u8
    
생성된 index.m3u8은 다음과 같다. ::

    #EXTM3U
    #EXT-X-TARGETDURATION: 10
    #EXT-X-MEDIA-SEQUENCE: 0
    #EXTINF:10,
    /video.mp4/mp4hls/0.ts
    #EXTINF:10,
    /video.mp4/mp4hls/1.ts
    #EXTINF:10,
    /video.mp4/mp4hls/2.ts
    
    ... (생략)...
    
    #EXTINF:10,
    /video.mp4/mp4hls/161.ts
    #EXTINF:9,
    /video.mp4/mp4hls/162.ts
    #EXT-X-ENDLIST
    
#EXT-X-TARGETDURATION은 ``<Duration>`` 으로 설정할 수 있다. 
주의할 점은 원본파일은 정확히 Video의 KeyFrame에 의해서만 분할된다는 것이다. 
다음 4가지 경우가 존재할 수 있다.

-  **KeyFrame 간격보다 ``<Duration>`` 설정이 큰 경우**
   
   KeyFrame이 3초, ``<Duration>`` 이 20초라면 20초를 넘지 않는 KeyFrame의 배수인 18초로 분할된다.
   
-  **KeyFrame 간격과** ``<Duration>`` **이 비슷한 경우**
   
   KeyFrame이 9초, ``<Duration>`` 이 10초라면 10초를 넘지 않는 KeyFrame의 배수인 9초로 분할된다.
   
-  **KeyFrame 간격이** ``<Duration>`` **설정보다 큰 경우**

   KeyFrame단위로 분할된다.
   
-  **Video가 없는 경우**

   ``<Duration>`` 단위로 분할된다.
   
#EXT-X-MEDIA-SEQUENCE은 .ts파일의 시작 숫자를 정의하며 ``<Sequence>`` 로 설정한다.

다음 클라이언트 요청에 대해 STON이 어떻게 동작하는지 이해해보자. ::

    GET /video.mp4/mp4hls/99.ts HTTP/1.1
    Range: bytes=0-512000
    Host: www.wineosft.com

1.	``STON`` 최초 로딩. (아무 것도 캐싱되어 있지 않음.)
#.	``Client`` HTTP Range 요청. (100번째 파일의 최초 500KB 요청)     
#.	``STON`` /video.mp4 파일 캐싱객체 생성.
#.	``STON`` /video.mp4 파일 분석을 위해 필요한 부분만을 원본서버에서 다운로드
#.	``STON`` 100번째(99.ts)파일 서비스를 위해 필요한 부분만을 원본서버에서 다운로드
#.	``STON`` 100번째(99.ts)파일 생성 후 Range 서비스
#.	``STON`` 서비스가 완료되면 99.ts파일 파괴