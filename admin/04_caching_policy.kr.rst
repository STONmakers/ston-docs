.. _caching_policy:

캐싱정책
******************

Caching-Key와 TTL(Time To Live)에 대해 설명한다.

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


Accept-Encoding 헤더
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

예외조건이 ``<ApplyQueryString>`` 설정에 따라 의미가 달라짐에 주의한다. 
명확한 URL또는 패턴(*만 허용한다)으로 설정이 가능하다.


Vary 헤더
---------------------

원본서버의 Vary헤더를 인식하여 콘텐츠를 구분한다. 일반적으로 Vary헤더는 Cache서버의 
성능을 급격히 떨어트리기 때문에 활성화되어 있지 않다. ::

    <Options>
        <VaryHeader />
    </Options>

예를 들어 원본서버가 다음과 같이 Vary헤더를 보냈다고 하더라도 
인식할 헤더가 설정되어 있지 않다면 무시된다. ::

    Vary: Accept-Encoding, Accept, User-Agent

User-Agent를 제외한 Accept-Encoding과 Accept헤더만을 인식하도록 하려면 
다음과 같이 설정한다. 구분자는 Comma(,)를 사용한다. ::

    <Options>
        <VaryHeader>Accept-Encoding, Accept</VaryHeader>
    </Options>
    
원본서버가 보낸 모든 Vary헤더를 인식하게 하려면 다음과 같이 설정한다. ::

    <Options>
        <VaryHeader>*</VaryHeader>
    </Options>




TTL (Time To Live)
====================================

TTL이란 원본으로부터 저장된 콘텐츠의 유효시간을 의미한다. 
TTL을 길게 설정하면 원본서버의 부하는 줄어들지만 변경사항이 늦게 반영된다. 
반대로 짧게 설정하면 너주 잦은 변경확인 요청으로 원본서버 부하가 높아진다.
Cache운영의 묘미는 TTL을 활용하여 원본서버 부하를 줄이는 것에 있다.

기본설정
---------------------

기본적으로 원본서버의 응답에 따라 TTL이 결정된다. 
TTL이 만료되기 전까지 저장된 콘텐츠로 서비스 된다.
TTL이 만료되면 원본서버로 콘텐츠 변경여부(If-Modified-Since 또는 If-None-Match)를 확인한다.
원본서버가 304 Not Modified응답을 준다면 TTL은 연장된다. ::

    <Options>
        <TTL>
            <NoCache Ratio="0" Max="5" MaxAge="0">5</NoCache>
            <Res2xx Ratio="20" Max="86400">1800</Res2xx>
            <Res3xx>300</Res3xx>
            <Res4xx>30</Res4xx>
            <Res5xx>30</Res5xx>
            <ConnectTimeout>3</ConnectTimeout>
            <ReceiveTimeout>3</ReceiveTimeout>
            <OriginBusy>3</OriginBusy>
        </TTL>
    </Options>
    
``Ratio`` (0~100)를 제외한 모든 설정 단위는 초(sec) 다.

-  ``<NoCache> (기본: 5초, Ratio: 0, Max=5, MaxAge=0)``
    원본서버가 no-cache로 응답했을 때 TTL을 설정한다. ::
    
    cache-control: no-cache 또는 private 또는 must-revalidate
    
    콘텐츠를 처음 저장할 때 ``<NoCache>`` 뒤에 콘텐츠가 만료(TTL)되도록 설정한다.
    (TTL만료 후) 원본서버에서 변경되지 않았다면(304 Not Modified) ``Ratio`` 비율(0~100)만큼 TTL을 연장한다.
    TTL은 최대 ``Max`` 까지 증가한다.
    
    .. note::

       ``<NoCache>`` 값 0으로 설정하면 서비스 직후 TTL이 곧바로 만료된다.
       만약 모든 요청에 대해 원본서버의 응답을 주고 싶다면, 바이패스할 것을 권장한다.     
    

-  ``<Res2xx>``

-  ``<Res3xx>``

-  ``<Res4xx>``

-  ``<Res5xx>``

-  ``<ConnectTimeout>``

-  ``<ReceiveTimeout>``

-  ``<OriginBusy>``

   
   
   