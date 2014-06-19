.. STON documentation master file, created by
   sphinx-quickstart on Fri Jun 13 16:37:06 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

STON Web Cache 소개
================================

Contents:

.. toctree::
   :maxdepth: 1

   admin/index.kr
   business/index.kr

   
Web Cache
---------
Web Cache는 클라이언트에 가장 가까운 (Edge) 위치에서 컨텐츠를 전달하는 서버의 한 종류로서 다음과 같이 동작하는 특징을 가진다.
클라이언트로부터 컨텐츠의 최초 요청을 받고
컨텐츠를 원본서버에서 가져와 저장하여 클라이언트에게 다시 전송한다. 
컨텐츠 재요청을 받으면 원본에서 가져왔던 즉시 전송한다. 

다양한 통신과 연산을 요구받는 일반적인 웹서버와 달리, Web Cache는 정적 컨텐츠 전송에 특화되어 다수 클라이언트에게 동일 컨텐츠를 전송하는 서비스에 매우 효과적이다.
   
.. image:: ston.png

서비스 확장 때문에 원본 웹서버를 Scale-Out 하면 컨텐츠 갱신마다 동기화 시켜야 하는 문제가 있다. 동일 컨텐츠를 수백대의 웹서버에 주입한다는 것은 리스크도 크고 관리도 어렵다. 그러나 Web Cache를 사용하면 원본 웹서버는 그대로 유지하면서 Web Cache를 Scale-Out하는것 만으로 손쉽게 서비스를 확장할 수 있다. 원본 웹서버의 컨텐츠만 갱신하면, Web Cache가 자동으로 컨텐츠를 가져다가 전송한다.


STON: SE의 Web Cache
--------------------

STON은 초고속 Web Cache 소프트웨어 솔루션으로서 기획단계부터 서비스 운영자 (SE, System Engineer)의 관점에 맞추어 설계되었다. 별도의 언어를 습득할 필요도 없으며, 설정변경을 위해 빌드를 다시 할 필요도 없다.


기능과 철학
-----------

STON의 구조는 최대한 많은 상황과 요구사항을 담을 수 있는 커다란 그릇을 만들고자 설계되었다. 그래서 기존의 오픈소스 솔루션들은 전혀 참조 대상이 아니었으며, 프로토콜 처리부분부터 한줄씩 모두 in-house로 제작되었다. 

STON의 개념, 동작방식, 배경, 이유는 홈페이지에 상세하게 공개되고 있으며, 좋은 운영자가 되고자 하는 많은 이들이 STON의 홈페이지를 통하여 지식을 습득하고 있다.

.. image:: features.png

STON은 사용에 숙련도가 필요없는 제품을 목표로 하고 있다.

손쉬운 사용
+++++++++++

STON은 Zero-Configuration을 추구하고 있다. 
