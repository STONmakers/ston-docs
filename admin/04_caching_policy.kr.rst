.. _caching_policy:

캐싱정책
******************

Caching-Key의 개념과 컨텐츠 만료 정책에 대해 설명한다.
원본서버의 부하를 줄이기 위해서는 캐싱정책을 효과적으로 설정해야 한다.

.. note::

   설명되는 설정을 모든 가상호스트의 기본 설정으로 적용하고 싶다면 
   <VHostDefault>태그 하위에, 해당 가상호스트에만 적용하고 싶다면 
   <Vhost>태그 하위에 설정한다.



.. toctree::
   :maxdepth: 2

Caching-Key
====================================

Caching-Key란 원본서버로부터 저장된 콘텐츠를 구분하는 고유 값이다. 
모든 파일이 /usr/conf.txt 처럼 다른 파일들과 구분되는 고유 값을 가지는 것과 같은 개념이다.
흔히 Caching-Key는 URL과 혼돈하기 쉽다.
하지만 같은 URL이라고 하더라도 HTTP의 여러 요소들에 따라 콘텐츠가 달라질 수 
있으므로 주의해야 한다.
같은 URL인데 서로 다른 Caching-Key를 사용할 수 있고, 반대인 경우도 있다.


Accept-Encoding
---------------------

같은 URL에 대한 HTTP요청이라도 Accept-Encoding헤더의 존재 유무에 따라 
같은(또는 다른) 컨텐츠가 캐싱될 수 있다. 원본서버에 요청을 보내는 시점에 
STON은 원본서버의 압축여부를 알 수 없다.

   .. figure:: img/acceptencoding.png
      :align: center

   원본서버가 어떤 응답을 줄지 알 수 없다. 

::

    <Options>
        <AcceptEncoding>ON</AcceptEncoding>
    </Options>

-  ``<AcceptEncoding>``

   -  ``ON (기본)`` HTTP 클라이언트가 보내는 Accept-Encoding 헤더를 인식한다.
   
   -  ``OFF`` HTTP 클라이언트가 보내는 Accept-Encoding 헤더를 무시한다.
    
원본서버에서 압축을 지원하지 않거나, 압축이 필요없는 대용량 파일의 경우 
OFF로 설정하는 것이 바람직하다.


대소문자 구분
---------------------

원본서버가 대소문자를 구분하지 않거나, 파일명이 소문자로만 구성된 경우 
대소문자를 구분하지 않도록 설정한다.

   .. figure:: img/casesensitive.png
      :align: center

   아마도 같은 콘텐츠이거나 404가 발생한다.
   
::

    <Options>
        <CaseSensitive>ON</CaseSensitive>
    </Options>

-  ``<CaseSensitive>``

   -  ``ON (기본)`` URL 대소문자를 구문한다. 
   
   -  ``OFF`` URL 대소문자를 구분하지 않는다. 모두 소문자로 처리된다.
    
QueryString 구분
---------------------

QueryString에 의하여 동적으로 생성되는 컨텐츠가 아니라면 QueryString을 인식하는 것은 
불필요하다. 최악의 경우로 QueryString이 아무 의미없는 Random이나 시간 값으로 증가하는 
경우에는 원본에 엄청난 부하가 발생할 수 있다.

   .. figure:: img/querystring.png
      :align: center

   동적 콘텐츠가 아니라면 같은 콘텐츠일 가능성이 높다.
   
::

    <Options>
        <ApplyQueryString>ON</ApplyQueryString>
    </Options>

-  ``<ApplyQueryString>``

   -  ``ON (기본)`` QueryString을 인식한다. 예외조건에 만족하면 QueryString이 무시된다.
   
   -  ``OFF`` QueryString을 무시한다. 예외조건에 만족하면 QueryString을 인식한다.
    
QueryString-예외조건은 /svc/{가상호스트 이름}/querystring.txt에 설정한다. ::

    # ./svc/www.example.com/querystring.txt
    /private/personal.jsp?login=ok*
    /image/ad.jpg

예외조건이 
``<ApplyQueryString>``설정에 따라 의미가 달라짐에 주의한다. 
명확한 URL또는 패턴(*만 허용한다)으로 설정이 가능하다.