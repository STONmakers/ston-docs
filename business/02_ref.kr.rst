.. _ref:

이용 및 고객사
=======

쇼핑몰 
------
**수많은 파일의 전송**

.. image:: /img/ref/shopping.png
  :align: center
  
고객은 구매결정에 이르기까지 썸네일, 전면배치 사진, 상세 사진 등 다양한 이미지를 보게 된다.
오픈마켓에는 하루에도 수천 수만 개의 상품이 등록되었다가 사라지는데, 
상품이미지들은 모두 제각각 유효기간이 다를 수도 있다. 
소셜커머스는 Time-Critical한 서비스이기 때문에, 이미지를 포함한 컨텐츠의 유효성 관리가 매우 중요하다.
STON은 정밀한 TTL (Time-To-Live) 설정을 제공하여 컨텐츠 유효성 관리를 손쉽게 해줄 뿐만 아니라, 
DIMS (Dynamic Image Management Service) 설정에 따라 상품 이미지를 On-the-fly로 가공하여 전송할 수 있다. 
(리사이즈, 크롭, 포맷변경, 조합 등) 또한 원본 장애시에도 설정에 따라 고객에게 상품페이지를 원활하게 전달한다.

게임
------
**대용량 파일 전송**

.. image:: /img/ref/game.png
  :align: center

온라인 게임은 클라이언트 파일을 빠르고 결함없이 배포해야 한다. 
고사양화로 클라이언트 파일 용량은 계속 증가하여 수십기가에 이르기도 한다.
STON은 서버/네트워크 자원활용을 극한까지 끌어올려 빠르게 전송한다.
그리고 파일을 이어 받고자 할 때, 정확한 부분전송으로 효율성을 높이고 전송시간을 단축시킨다.

언론 / 커뮤니티
-----------------
**번개같은 응답속도**

.. image:: /img/ref/news.png
  :align: center

언론사, 커뮤니티 사이트는 동일 컨텐츠를 많은 사용자들에게 제공하기 때문에 304 Not Modified 
응답의 비율이 높은 편이다. 
304 응답의 효율성은 자체의 크기보다 반응속도에 큰 영향을 받는데, 
STON은 초고속 응답속도로 캐싱효율을 배가시키는 효과를 일으킨다.
또한 이미지 서비스의 중요성이 더욱 커지고 다양한 디바이스들이 보급되면서, 
이미지 관리 및 커스터마이징에 대한 필요성이 계속 요구되고 있다. 
STON은 DIMS (Dynamic Image Management Service) 를 제공하여 이미지를 설정에 따라
자동가공하여 전송하는 기능을 제공하고 있다. 

동영상 서비스
-------------
**HTTP 동영상 전송**

.. image:: /img/ref/media.png
  :align: center

미디어 전용 프로토콜의 사용은 줄어들고 있는 반면, HTTP/MP4 동영상 서비스는 점점 늘어가고 있다.
특히 모바일 디바이스의 급속한 보급으로 HTTP 기반의 Streaming 방식이 점점 보편화 되는 추세다.
STON은 HLS (HTTP Live Streaming)을 지원하여, 헤더제어를 이용한 원활한 Pseudo-Streaming, 
대역폭 조절로 다양한 Bitrate 재생과, 사용자가 원하는 구간추출재생 등의 기능을 제공하고 있다.


Customers/Partners
------------------

.. image:: img/customers.png
  :align: center

Powered by STON
---------------

.. image:: img/poweredby.png
  :align: center


이용문의: license@winesoft.co.kr
