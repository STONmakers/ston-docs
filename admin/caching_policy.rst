.. _caching-policy:

4장. Caching 정책
******************

이 장에서는 서비스의 핵심이 되는 TTL(Time To Live)과 Caching-Key 그리고 만료정책에 대해 설명한다.
저장된 콘텐츠는 TTL동안 유효하다.
HTTP 규격은 TTL을 설정할 수 있도록 Cache-Control을 명시하고 있다.
하지만 이는 절대적인 것은 아니다. 
다양한 방식의 TTL 정책과 :ref:`caching-purge` 를 통해 서비스 품질을 높일 수 있다.

HTTP에는 콘텐츠를 구분하는 다양한 규격이 존재한다. 
그만큼 Caching-Key도 다양하게 존재할 수 있다.
콘텐츠 변경이 없을수록 원본부하를 줄일 수 있을뿐만 아니라 쉽게 확장할 수 있다.
서비스에 최적화된 만료정책을 수립하는 다양한 방식에 대해 설명한다.

앞으로 설명되는 설정을 모든 가상호스트의 기본 설정으로 적용하고 싶다면 ``<VHostDefault>`` 하위에 설정한다.
반대로 특정 가상호스트에만 적용하고 싶다면 <Vhost>태그 하위에 설정한다.

**Caching-Key**란 콘텐츠를 구분하는 고유 값이다. 
파일시스템에서 파일들과 구분되는 고유경로(예. /usr/conf.txt)를 가지는 것과 같은 개념이다.
흔히 Caching-Key는 URL과 혼동되기 쉽다.
HTTP의 여러 기능에 따라 같은 URL이라고 하더라도 콘텐츠가 달라질 수 있다.


.. toctree::
   :maxdepth: 2



.. _caching-policy-ttl:

TTL (Time To Live)
====================================

TTL이란 저장된 콘텐츠의 유효시간이다.
TTL을 길게 설정하면 원본서버의 부하는 줄어들지만 변경사항이 늦게 반영된다. 
반대로 짧게 설정하면 너무 잦은 변경확인 요청으로 원본서버 부하가 높아진다.
운영의 묘미는 TTL을 적절히 설정하여 원본부하를 줄이는 것에 있다.
TTL은 한번 설정되면 만료되기 전까지 바뀌지 않는다.
새로운 TTL은 파일이 만료되었을 때 적용된다.
관리자는 :ref:`api-cmd-purge` , :ref:`api-cmd-expire` , :ref:`api-cmd-expireafter` , :ref:`api-cmd-hardpurge` 등의 API를 사용해 TTL을 변경할 수 있다.


기본 TTL
---------------------

기본적으로 TTL은 원본서버의 응답에 따라 결정된다. 
TTL이 만료되기 전까지 저장된 콘텐츠로 서비스 된다.
TTL이 만료되면 원본서버로 콘텐츠 변경여부( **If-Modified-Since** 또는 **If-None-Match** )를 확인한다.
원본서버가 **304 Not Modified** 응답을 준다면 TTL은 연장된다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <TTL>
        <Res2xx Ratio="20" Max="86400">1800</Res2xx>
        <NoCache Ratio="0" Max="5" MaxAge="0">5</NoCache>
        <Res3xx>300</Res3xx>
        <Res4xx>30</Res4xx>
        <Res5xx>30</Res5xx>
        <ConnectTimeout>3</ConnectTimeout>
        <ReceiveTimeout>3</ReceiveTimeout>
        <OriginBusy>3</OriginBusy>
    </TTL>    
    
``Ratio`` (0~100)를 제외한 모든 설정 단위는 초(sec) 다.

-  ``<Res2xx> (기본: 1800초, Ratio: 20, Max=86400)``
   원본서버가 200 OK로 응답했을 때 TTL을 설정한다.
   콘텐츠를 처음 저장할 때 ``<Res2xx>`` 초 뒤에 콘텐츠가 만료(TTL)되도록 설정한다.
   (TTL만료 후) 원본서버에서 변경되지 않았다면(304 Not Modified) ``Ratio`` 비율(0~100)만큼 TTL을 연장한다.
   TTL은 최대 ``Max`` 까지 증가한다.

-  ``<NoCache> (기본: 5초, Ratio: 0, Max=5, MaxAge=0)``
   ``<Res2xx>`` 와 동일하나 원본서버가 no-cache로 응답하는 경우에만 적용된다. ::
   
      cache-control: no-cache 또는 private 또는 must-revalidate
    
   ``MaxAge`` 가 0보다 크다면 max-age를 줄 수 있다.
    
   .. figure:: img/nocache_maxage.png
      :align: center

      Max-Age만큼 클라이언트에 Caching된다.

-  ``<Res3xx> (기본: 300초)``
   원본서버가 3xx로 응답했을 때 TTL을 설정한다. 
   Redirect용도로 사용되는 경우가 많다.

-  ``<Res4xx> (기본: 30초)``
   원본서버가 4xx로 응답했을 때 TTL을 설정한다. 
   **404 Not Found** 인 경우가 많다.

-  ``<Res5xx> (기본: 30초)``
   원본서버가 5xx로 응답했을 때 TTL을 설정한다. 
   원본서버 내부 장애상황인 경우가 많다.

-  ``<ConnectTimeout> (기본: 3초)``
   원본서버로 접속하지 못하는 경우 TTL을 설정한다.
   콘텐츠가 이미 저장되어 있다면 ``<ConnectTimeout>`` 초 만큼 TTL을 연장한다.
   콘텐츠가 저장되어 있지 않다면 ``<ConnectTimeout>`` 초 만큼 장애상황으로 응답한다.
   이는 장애상황을 서비스한다는 의미보다는 TTL시간동안 (아마도 장애상황일) 원본서버에 부담을 주지 않기 위함이다.

-  ``<ReceiveTimeout> (기본: 3초)``
   접속은 됐으나 데이터를 수신하지 못하는 경우 TTL을 설정한다.
   ``<ConnectTimeout>`` 과 의미적 동일하다.

-  ``<OriginBusy> (기본: 3초)``
   :ref:`origin-busysessioncount` 조건을 만족하면 원본서버 요청없이 만료된 콘텐츠의 TTL을 설정된 시간만큼 연장한다.
   이는 원본서버의 부하를 가중시키지 않기 위함이다.
   
.. note::

   TTL 값을 0으로 설정하면 서비스 직후 곧바로 만료된다.
   만약 모든 요청에 대해 원본서버의 응답을 주고 싶다면 바이패스할 것을 권장한다.
   

.. _caching-policy-customttl:
   
Custom TTL
---------------------

URL마다 별도로 TTL을 설정한다.
명확한 URL 또는 패턴 URL에 매칭되는 콘텐츠마다 고정된 TTL을 설정할 수 있다.
/svc/{가상호스트 이름}/ttl.txt 에 설정한다. ::

    # /svc/www.example.com/ttl.txt
    # 구분자는 콤마(,)이며 시간 단위는 초이다.
    
    *.jsp, 10
    /,5
    /index.html, 5
    /script/*.js, 300
    /image/ad.jpg, 1800
    

모든 페이지(html, php, jsp 등)에 별도의 TTL을 설정하기 위하여 *.html을 추가하였더라도 첫 페이지(/)에는 설정되지 않는다. 
원본서버가 첫 페이지를 어떤 페이지(예를 들어 index.php로 default.jsp 등)로 설정하였는지 HTTP 프로토콜로는 알 수 없다. 
그러므로 모든 페이지에 별도의 TTL을 설정하려면 반드시 /를 추가해야 한다.
    
    
TTL 우선순위
---------------------

적용할 TTL설정의 우선순위를 설정한다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <TTL Priority="cc_nocache, custom, cc_maxage, rescode">
        ... (생략) ...
    </TTL>    
    
``<TTL>`` 의 ``Priority (기본: cc_nocache, custom, cc_maxage, rescode)`` 속성으로 설정한다.

- ``cc_nocache`` 원본이 Cache-Control: no-cache로 응답한 경우
- ``custom`` `caching-policy-customttl`
- ``cc_maxage`` 원본이 Cache-Control에 maxage를 명시한 경우
- ``rescode`` 원본 응답코드별 기본 TTL


비정상 TTL 연장
---------------------

원본서버 종료로 인해 응답이 오지 않는 경우에는 장애판단이 명확하지만 간혹 정상적으로 응답하면서 장애상황인 경우가 발생한다.
예를 들어 콘텐츠를 저장하는 Storage와의 연결을 잃거나, 뭔가 정상처리가 불가능하다고 판단하는 경우가 있을 수 있다.
전자의 경우 4xx응답(주로 **404 Not Found** ), 후자는 5xx응답(주로 **500 Internal Error** )을 받게된다.

하지만 이미 관련 콘텐츠가 저장되어 있다면, 
원본의 응답을 믿는 것보다 TTL을 연장시켜 서비스 전체장애가 발생하지 않도록 하는편이 효과적이다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <TTLExtensionBy4xx>OFF</TTLExtensionBy4xx>
    <TTLExtensionBy5xx>ON</TTLExtensionBy5xx>

-  ``<TTLExtensionBy4xx>``

   -  ``OFF (기본)`` 4xx 응답으로 콘텐츠를 갱신한다.
   
   -  ``ON`` 304 not modified를 받은 것처럼 동작한다.
   
의도된 4xx응답이 아닌지 주의해야 한다.
       
-  ``<TTLExtensionBy5xx>``

   -  ``ON (기본)`` **304 Not Modified** 를 받은 것처럼 동작한다.
   
   -  ``OFF`` 5xx 응답으로 콘텐츠를 갱신한다.

정상적인 서버라면 5xx로 응답하지 않는다. 
주로 서버의 일시적인 장애로부터 콘텐츠를 무효화하여 원본부하를 가중시키지 않기 위한 용도로 사용된다.
    

.. _caching-policy-renew:

갱신정책
====================================

TTL이 만료된 콘텐츠의 경우 원본서버에서 갱신여부를 확인한 뒤 서비스가 이루어진다.

   .. figure:: img/perf_refreshexpired.jpg
      :align: center
      
      변경확인 후 응답

1. TTL이 유효하다. 
   즉시 응답한다.

#. TTL이 만료되어 원본서버로 변경확인(If-Modified-Since)을 요청한다. 
   변경확인이 될때까지 클라이언트에게 응답하지 않는다.
   
#. 원본서버에서 응답이 오면 TTL을 연장하거나 콘텐츠를 변경(Swap)한다. 
   원본서버에서 확인이 되었으므로 클라이언트에게 응답한다.
   
#. 변경확인이 된 콘텐츠이므로 다음 TTL 만료시까지 즉시 응답한다.

고화질 동영상이나 게임처럼 상대적으로 반응성보다 전송속도가 중요한 서비스에서는 이런 방식이 큰 문제가 되지 않는다. 
대용량 데이터의 경우 원본서버가 10초만에 갱신에 대한 응답을 한다고 하더라도 전송에 몇 분씩 소요되기 때문에 상대적으로 원본의 반응성이 크게 중요하지 않다. 
오히려 접근 빈도가 높지 않은 콘텐츠이기 때문에 반드시 갱신확인이 이루어져야 한다.

하지만 쇼핑몰과 같은 경우 상황은 다르다. 
웹 페이지는 빠르게 로딩되는 것이 무엇보다 중요하다. 
1~2초 안에 클라이언트 화면구성이 모두 끝나야 한다.
전송속도보다 반응속도가 더 중요하다는 말이다. 

이때 TTL이 만료되어 원본서버에게 갱신확인을 해야 한다면 매우 큰 지연이 발생할 수 있다. 
보통의 쇼핑몰이 수백만개의 콘텐츠를 동시에 서비스 하는 것을 감안할 때 항상 원본서버로부터 갱신확인 작업이 발생하고 있다고 생각해야 한다. 
자칫 원본서버 장애가 발생하거나 네트워크 장애가 발생한다면 최악이다.

우리가 원하는 것은 어떠한 원본서버의 장애나 지연으로부터 캐싱된 콘텐츠가 안전하게 전송되는 것이다.

   .. figure:: img/perf_refreshexpired2.jpg
      :align: center
      
      장애가 두렵지 않다!
      
이런 차이점 때문에 백그라운드 콘텐츠 갱신기능이 개발되었다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <RefreshExpired>ON</RefreshExpired>    

-  ``<RefreshExpired>``

   -  ``ON (기본)`` 변경확인 후 응답한다.
   
   -  ``OFF`` 변경확인 응답을 기다리지 않고 응답한다.
      새로운 콘텐츠의 다운로드가 완료되면 그때 변경(Swap)한다.

``OFF`` 설정의 더 큰 이유는 콘텐츠가 대부분 자주 바뀌지 않기 때문이다.

   .. figure:: img/perf_refreshexpired5.jpg
      :align: center
      
      변경에 민감하지 않다면 기다리지 않는다.

위 그림에서 원본서버와의 갱신작업이 모두 백그라운드로 이루어지기 때문에 캐싱된 콘텐츠는 기다림없이 즉시 클라이언트에게 서비스된다. 
원본서버가 **304 Not Modified** 로 응답한다면 TTL만 연장된다. 
파일이 갱신되어 원본서버에서 200 OK를 응답한 경우 해당 파일이 완전히 다운로드된 후에 파일이 부드럽게 교체된다. 
콘텐츠가 바뀌어도 이전 콘텐츠(초록색)를 다운로드 받던 사용자들은 정상적으로 다운로드가 진행된다. 
파일 교체 이후 접근된 사용자들(겨자색)은 바뀐 파일로 서비스 된다.
콘텐츠 갱신, 네트워크 장애, 원본서버 장애 등 어떠한 변수에도 콘텐츠 갱신은 백그라운드로 진행되기 때문에 실제 서비스에는 전혀 지연이 없다.


클라이언트 no-cache 요청시 TTL만료
---------------------

클라이언트 HTTP요청에 no-cache 설정이 하나 이상 명시된 경우 해당 콘텐츠를 즉시 만료시킬 수 있다. ::

    GET /logo.jpg HTTP/1.1
    ...
    cache-control: no-cache 또는 cache-control:max-age=0
    pragma: no-cache
    ...
    
::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <NoCacheRequestExpire>OFF</NoCacheRequestExpire>    

-  ``<NoCacheRequestExpire>``

   -  ``OFF (기본)`` 무시한다.
   
   -  ``ON`` TTL을 즉시 만료한다.
   
만료된 콘텐츠는 `갱신정책`_ 에 따른다.


.. _caching-policy-accept-encoding:

Accept-Encoding 헤더
====================================

같은 URL에 대한 HTTP요청이라도 Accept-Encoding헤더의 존재 유무에 따라 다른 콘텐츠가 캐싱될 수 있다. 
원본서버에 요청을 보내는 시점에 압축여부를 알 수 없다.
응답을 받았다고해도 압축여부를 매번 비교할 수도 없다.

   .. figure:: img/acceptencoding.png
      :align: center

      원본서버가 어떤 응답을 줄지 알 수 없다.

::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <AcceptEncoding>ON</AcceptEncoding>    

-  ``<AcceptEncoding>``

   -  ``ON (기본)`` HTTP 클라이언트가 보내는 Accept-Encoding 헤더를 인식한다.
   
   -  ``OFF`` HTTP 클라이언트가 보내는 Accept-Encoding 헤더를 무시한다.
    
원본서버에서 압축을 지원하지 않거나, 압축이 필요없는 대용량 파일의 경우 ``OFF`` 로 설정하는 것이 바람직하다.


.. _caching-policy-casesensitive:

대소문자 구분
====================================

원본서버의 대소문자 구분여부를 능동적으로 알 수 없다.

   .. figure:: img/casesensitive.png
      :align: center

      아마도 같은 콘텐츠이거나 404가 발생한다.
   
::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <CaseSensitive>ON</CaseSensitive>    

-  ``<CaseSensitive>``

   -  ``ON (기본)`` URL 대소문자를 구문한다. 
   
   -  ``OFF`` URL 대소문자를 구분하지 않는다. 모두 소문자로 처리된다.

    
.. _caching-policy-applyquerystring:
    
QueryString 구분
====================================

QueryString에 의하여 동적으로 생성되는 콘텐츠가 아니라면 QueryString을 인식하는 것은 불필요하다. 
아무 의미없는 Random값이나 항상 변하는 시간 값이 매번 붙는다면 원본에 엄청난 부하가 발생할 수 있다.

   .. figure:: img/querystring.png
      :align: center

      동적 콘텐츠가 아니라면 같은 콘텐츠일 가능성이 높다.
   
::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <ApplyQueryString Collective="OFF">ON</ApplyQueryString>

-  ``<ApplyQueryString>``

   -  ``ON (기본)`` QueryString을 인식한다. 예외조건에 만족하면 QueryString이 무시된다.
   
   -  ``OFF`` QueryString을 무시한다. 예외조건에 만족하면 QueryString을 인식한다.
    
QueryString-예외조건은 /svc/{가상호스트 이름}/querystring.txt에 설정한다. ::

    # ./svc/www.example.com/querystring.txt
    
    /private/personal.jsp?login=ok*
    /image/ad.jpg

예외조건이 ``<ApplyQueryString>`` 설정에 따라 의미가 달라짐에 주의한다. 
명확한 URL또는 패턴(*만 허용한다)으로 설정이 가능하다.

``Collective`` 속성은 :ref:`caching-purge` API가 호출되었을 때 대상을 지정한다.

-  ``Collective``

   -  ``OFF (기본)`` 파라미터 URL만을 대상으로 지정한다.
   
   -  ``ON`` 파라미터 URL뿐만 아니라 URL에 QueryString이 존재하는 모든 컨텐츠를 대상으로 지정한다.
   
``Collective`` 속성이 ON이고 파일이 많을수록 :ref:`caching-purge` 에 CPU부하가 높아진다. 
관련 파일을 검색하는 소요시간이 길어질 수 있어 예기치 않은 문제를 일으킬 수 있다.
가급적 QueryString까지 붙은 명확한 URL로 :ref:`caching-purge` API를 호출할 것을 권장한다.




Vary 헤더
====================================

Vary헤더를 인식하여 콘텐츠를 구분한다. 
일반적으로 Vary헤더는 Cache서버의 성능을 급격히 떨어트리는 원흉이다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <VaryHeader />    
    
-  ``<VaryHeader>``

   원본서버가 응답한 Vary헤더 중 지원할 헤더목록을 설정한다.
   구분자는 콤마(,)를 사용한다.

예를 들어 원본서버가 다음과 같이 Vary헤더를 보냈다고 하더라도 ``<VaryHeader>`` 가 설정되어 있지 않다면 무시한다. ::

    Vary: Accept-Encoding, Accept, User-Agent

User-Agent를 제외한 Accept-Encoding과 Accept헤더만을 인식하도록 하려면 다음과 같이 설정한다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <VaryHeader>Accept-Encoding, Accept</VaryHeader>    
    
원본서버가 보낸 모든 Vary헤더를 인식하게 하려면 다음과 같이 설정한다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <VaryHeader>*</VaryHeader>    


.. _caching-policy-post-method-caching:

POST 요청 캐싱
====================================

POST 요청을 Caching하도록 설정한다. 
POST 요청의 특성상 URL은 같지만 Body데이터가 다를 수 있다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>
    
    <PostRequest MaxContentLength="102400" BodySensitive="ON">OFF</PostRequest>    

-  ``<PostRequest>``

   -  ``OFF (기본)`` POST요청이 오면 세션을 종료한다.
   
   -  ``ON`` POST요청을 Caching한다.
   
실제로 POST요청을 처리하는 대부분의 경우는 Body데이터를 Caching-Key로 사용한다.
``BodySensitive`` 속성과 예외조건을 통해 정교한 설정이 가능하다.

-  ``BodySensitive``

    -  ``ON (기본)`` Body데이터까지 Caching-Key로 인식한다.
       최대 길이는 ``MaxContentLength (기본: 102400 Bytes)`` 속성으로 제한한다.
       예외조건에 만족하면 Body데이터를 무시한다.
    
    -  ``OFF`` Body데이터는 무시한다. 
       예외조건에 만족하면 Body데이터를 인식한다.
   
POST요청 예외조건은 /svc/{가상호스트 이름}/postbody.txt에 설정한다. ::
    
    # /svc/www.example.com/postbody.txt
    
    /bigsale/*.php?nocache=*
    /goods/search.php
    
예외조건이 ``BodySensitive`` 설정에 따라 의미가 달라짐에 주의한다. 
명확한 URL 또는 패턴(*만 허용한다.)으로 설정이 가능하다.

이 설정은 :ref:`bypass-getpost` 와 정책적으로 혼란스러울 수 있다.
``<BypassPostRequest> (기본: ON)`` 에 의해 POST요청이 캐싱되지 않을 수 있다.
따라서 POST요청을 캐싱하기 위해서는 ``<BypassPostRequest>`` 를 ``OFF`` 또는 예외조건을 설정해야 한다.
정리하면 우선순위는 다음과 같다.
        
* 바이패스 조건( :ref:`bypass-getpost` )에 만족할 경우 원본서버로 바이패스 한다.
* Content-Length헤더가 없다면 연결을 종료한다.
* ``PostRequest`` 가 ``ON`` 으로 설정되어 있고 Content-Length가 ``MaxContentLength`` 속성 값을 넘지 않는다면 캐싱모듈에 의해 처리된다.
* 이상의 시나리오에서 처리되지 않은 요청은 종료한다.
  
.. note::

    ``MaxContentLength`` 속성을 너무 크게 설정할 경우 Caching-Key 관리에 많은 메모리가 필요하다.
    가능한 작게 설정하는 것이 좋다.
    