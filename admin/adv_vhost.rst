.. _adv-vhost:

14장. 가상호스트 고급기법
******************

이 장에서는 가상호스트를 활용하여 서비스를 유연하게 구성하는 여러 기법에 대해 설명한다.

가상호스트는 보통 원본(Domain 또는 IP목록)과 1:1로 구성되는 것이 기본이다.
하지만 상황에 따라 대표 가상호스트를 여러 하위 가상호스트로 분기하거나,
반대로 독립적인 여러 가상호스트를 하나의 서비스로 포장해야 하는 경우도 발생한다.
각 기능에 따라 :ref:`monitoring_stats_vhost_client` / :ref:`admin-log-access` 의 정책이 다를 수 있음을 유의해야 한다.


.. toctree::
   :maxdepth: 2


.. _adv-vhost-url-rewrite:

URL 전처리
====================================

`정규표현식 <http://en.wikipedia.org/wiki/Regular_expression>`_ 을 사용하여 요청된 URL을 변경한다.
URL전처리가 설정되어 있다면 모든 클라이언 요청(HTTP 또는 File I/O)은 반드시 URL Rewriter를 거친다.

.. figure:: img/urlrewrite1.png
   :align: center

   URL Rewriter를 통과해야 가상호스트에 갈 수 있다.

만약 URL Rewriter에 의해 접근하려는 Host이름이 변경되었다면 클라이언트 HTTP요청의 Host헤더가 변경된 것으로 간주한다.
URL 전처리는 가상호스트 설정(vhosts.xml)에 설정한다.
대부분의 설정이 가상호스트에 종속되지만, URL전처리의 경우 클라이언트가 요청한 Host의 이름을 변경할 수 있으므로 가상호스트와 같은 레벨로 설정한다. ::

   # vhosts.xml

   <Vhosts>
      <Vhost ...> ... </Vhost>
      <Vhost ...> ... </Vhost>
      <URLRewrite ...> ... </URLRewrite>
      <URLRewrite ...> ... </URLRewrite>
   </Vhosts>

멀티로 설정할 수 있으며 순차적으로 정규표현식 일치 여부를 비교한다. ::

   # vhosts.xml - <Vhosts>

   <URLRewrite AccessLog="Replace">
       <Pattern>www.exmaple.com/([^/]+)/(.*)</Pattern>
       <Replace>#1.exmaple.com/#2</Replace>
   </URLRewrite>

-  ``<URLRewrite>``

   URL전처리를 설정한다.
   ``AccessLog (기본: Replace)`` 속성은  Access로그에 기록될 URL을 설정한다.
   ``Replace`` 인 경우 변환 후 URL(/logo.jpg)을, ``Pattern`` 인 경우 변환 전
   URL(/baseball/logo.jpg)을 Access로그에 기록한다.

   -  ``<Pattern>`` 매칭시킬 패턴을 설정한다.
      한개의 패턴은 ( ) 괄호를 사용하여 표현된다.

   -  ``<Replace>`` 변환형식을 설정한다.
      일치된 패턴에 대해서는 #1, #2와 같이 사용할 수 있다. #0는 요청 URL전체를 의미한다.
      패턴은 최대 9개(#9)까지 지정할 수 있다.

처리량은 :ref:`monitoring_stats` 로 제공되며 :ref:`api-graph-urlrewrite` 으로도 확인할 수 있다.
URL전처리는 :ref:`media-trimming` , :ref:`media-hls` 등 다른 기능들과 결합하여 표현을 간결하게 만든다. ::

   # vhosts.xml - <Vhosts>

   <URLRewrite>
       <Pattern>example.com/([^/]+)/(.*)</Pattern>
       <Replace>example.com/#1.php?id=#2</Replace>
   </URLRewrite>
   // Pattern : example.com/releasenotes/1.3.4
   // Replace : example.com/releasenotes.php?id=1.3.4

   <URLRewrite>
       <Pattern>example.com/download/(.*)</Pattern>
       <Replace>download.example.com/#1</Replace>
   </URLRewrite>
   // Pattern : example.com/download/1.3.4
   // Replace : download.example.com/1.3.4

   <URLRewrite>
       <Pattern>example.com/img/(.*\.(jpg|png).*)</Pattern>
       <Replace>example.com/#1/STON/composite/watermark1</Replace>
   </URLRewrite>
   // Pattern : example.com/img/image.jpg?date=20140326
   // Replace : example.com/image.jpg?date=20140326/STON/composite/watermark1

   <URLRewrite>
       <Pattern>example.com/preview/(.*)\.(mp3|mp4|m4a)$</Pattern>
       <Replace><![CDATA[example.com/#1.#2?&end=30&boost=10&bandwidth=2000&ratio=100]]></Replace>
   </URLRewrite>
   // Pattern : example.com/preview/audio.m4a
   // Replace : example.com/audio.m4a?end=30&boost=10&bandwidth=2000&ratio=100

   <URLRewrite>
       <Pattern>example.com/(.*)\.mp4\.m3u8$</Pattern>
       <Replace>example.com/#1.mp4/mp4hls/index.m3u8</Replace>
   </URLRewrite>
   // Pattern : example.com/video.mp4.m3u8
   // Replace : example.com/video.mp4/mp4hls/index.m3u8

   <URLRewrite>
       <Pattern>example.com/(.*)_(.*)_(.*)</Pattern>
       <Replace>example.com/#0/#1/#2/#3</Replace>
   </URLRewrite>
   // Pattern : example.com/video.mp4_10_20
   // Replace : example.com/example.com/video.mp4_10_20/video.mp4/10/20

패턴표현에 XML의 5가지 특수문자( " & ' < > )가 들어갈 경우 반드시 <![CDATA[ ... ]]>로 묶어주어야 올바르게 설정된다.
:ref:`wm` 을 통해 설정할 때 모든 패턴은 CDATA로 처리된다.



.. _adv-vhost-facadevhost:

Facade 가상호스트
====================================

``<Alias>`` 는 가상호스트의 별명만을 추가하는 것이므로 통계와 로그가 분리되지 않는다.
가상호스트는 공유하지만 도메인에 따라 :ref:`monitoring_stats_vhost_client` 와 :ref:`admin-log-access` 를 분리하고 싶은 경우 Facade가상호스트를 설정한다.

.. figure:: img/adv_vhost_facade.png
   :align: center

   facade는 통계와 로그만 수집한다.

::

    # vhosts.xml - <Vhosts>

    <Vhost Name="example.com">
       ...
    </Vhost>

    <Vhost Name="another.com" Status="facade:example.com">
       ...
    </Vhost>

``Status`` 속성의 값을 ``facade:`` + ``가상호스트`` 로 설정한다.
예제의 경우 :ref:`monitoring_stats_vhost_client` 와 :ref:`admin-log-access` 는 example.com이 아닌 클라이언트가 요청한 도메인인 another.com으로 수집된다.



.. _adv-vhost-sub-path:

Sub-Path 지정
====================================

한 가상호스트에서 경로에 따라 다른 가상호스트가 처리하도록 설정할 수 있다.

.. figure:: img/adv_vhost_subpath.png
   :align: center

   통계/로그는 요청을 최종처리한 각각의 가상호스트에 기록된다.


::

   # vhosts.xml - <Vhosts>

   <Vhost Name="sports.com">
     <Sub Status="Active">
       <Path Vhost="baseball.com">/baseball/<Path>
       <Path Vhost="football.com">/football/<Path>
       <Path Vhost="photo.com">/*.jpg<Path>
     </Sub>
   </Vhost>

   <Vhost Name="baseball.com" />
   <Vhost Name="football.com" />
   <Vhost Name="photo.com" />

-  ``<Sub>`` 경로나 패턴이 일치하면 해당 요청을 다른 가상호스트로 보낸다.
   일치하지 않는 경우만 현재 가상호스트가 처리한다.

   - ``Status (기본: Active)`` Inactive인 경우 무시한다.

   -  ``<Path>`` 클라이언트가 요청한 URI와 경로가 일치하면 ``Vhost`` 로 해당 요청을 보낸다.
      값은 경로 또는 패턴만 가능하다. ::

         <Path Vhost="baseball.com">baseball<Path>
         <Path Vhost="photo.com">*.jpg<Path>

      위와 같이 입력해도 각각 /baseball/과 /*.jpg로 인식된다.

예를 들어 클라이언트가 다음과 같이 요청했다면 해당 요청은 가상호스트 football.com이 처리한다. ::

   GET /football/rank.html HTTP/1.1
   Host: sports.com


.. _adv-vhost-redirection-trace:

Redirect 추적
====================================

원본서버에서 Redirect계열(301, 302, 303, 307)로 응답하는 경우 Location헤더를 추적하여 콘텐츠를 요청한다.

   .. figure:: img/conf_redirectiontrace.png
      :align: center

      클라이언트는 Redirect여부를 모른다.

::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <RedirectionTrace>OFF</RedirectionTrace>

-  ``<RedirectionTrace>``

   - ``OFF (기본)`` 3xx 응답으로 저장된다.

   - ``ON`` Location헤더에 명시된 주소에서 콘텐츠를 다운로드 한다.
     형식에 맞지 않거나 Location헤더가 없는 경우에는 동작하지 않는다.
     무한히 Redirect되는 경우를 방지하기 위하여 1회만 추적한다.



.. _adv-vhost-link:

가상호스트 링크
====================================

콘텐츠가 여러 원본에 분산되어 있다면, 가상호스트 링크를 활용하여 콘텐츠가 통합되어 있는 것처럼 서비스가 가능하다.
특히 On-Premise에서 클라우드로 스토리지를 마이그레이션하거나, 스토리지의 용량, 비용 등의 이유로 콘텐츠가 분산되어 있는 환경에서 유용하다.

.. figure:: img/adv_vhost_link.png
   :align: center

   cloud.com에 없는 콘텐츠는 nas.com이 처리한다.

::

   # vhosts.xml - <Vhosts><Vhost>

   <VhostLink Condition="...">...</VhostLink>

-  ``<VhostLink>`` 요청을 위임할 가상호스트 이름. 콘텐츠에 대한 원본 응답이 ``Condition`` 을 만족하면 지정된 가상호스트로 요청을 위임한다. 단 하나만 설정할 수 있다.

   - ``Condition`` HTTP 응답코드/패턴(1xx, 2xx, 3xx, 4xx, 5xx), fail(원본에서 캐싱하지 못한 경우)

클라이언트 요청이 다른 가상호스트로 위임되더라도 :ref:`monitoring_stats_vhost_client` 와 :ref:`admin-log-access` 는 클라이언트가 접근한 가상호스트에 기록된다.

.. note::

   링크 관계에 있는 가상호스트 설정이 다를 경우 의도치 않게 동작할 수 있음을 주의한다.
   가상호스트 링크가 A(단순 캐싱) -> B(이미지 압축)로 맺어져 있다면,
   A에서 처리된 이미지는 압축되지 않지만 B에서 처리된 이미지는 압축된다.

예를 들어 nas.com의 콘텐츠를 cloud.com으로 이전 중일 경우, cloud.com에 없는(=404 Not Found) 콘텐츠에 대해서만 nas.com으로 요청을 보낼 수 있다.
아래의 경우 요청이 nas.com에 의해 처리되더라도 :ref:`monitoring_stats_vhost_client` 와 :ref:`admin-log-access` 는 cloud.com에 기록된다.

::

   # vhosts.xml - <Vhosts>

   // cloud.com에 없는(=404 Not Found) 콘텐츠는 nas.com에서 서비스한다.
   <Vhost Name="cloud.com">
     <VhostLink Condition="404">nas.com</VhostLink>
   </Vhost>

   <Vhost Name="nas.com">
   </Vhost>


:ref:`admin-log-access` 의 vhostlink 필드를 통해 클라이언트 요청이 어느 가상호스트에서 처리되었는지 알 수 있다.
"-" 는 요청이 링크되지 않았음을 의미하며 "nas.com" 은 해당 요청이 링크되어 nas.com에서 처리되었음을 의미한다. ::

    #Fields: date time s-ip cs-method cs-uri-stem ...(중략)... vhostlink
    2016.11.24 16:52:24 220.134.10.5 GET /web/h.gif ...(중략)... -
    2016.11.24 16:52:26 220.134.10.5 GET /favicon.ico ...(중략)... nas.com

링크가 여러 번 발생했다면 "+"를 구분자로 링크된 모든 가상호스트가 명시된다.
이 경우 가장 마지막에 위치한 가상호스트가 최종 요청을 처리한 가상호스트이다.

다음과 같이 여러 가상호스트를 다른 조건으로 링크할 수 있다.

::

   # vhosts.xml - <Vhosts>

   // 원본서버가 5xx로 응답했거나 캐싱하지 못했을 때(=fail) 해당 요청을 bar.com에게 위임한다.
   <Vhost Name="foo.com">
     <VhostLink Condition="5xx,fail">bar.com</VhostLink>
   </Vhost>

   // 원본서버가 4xx로 응답했을 때 해당 요청을 helloworld.com에게 위임한다.
   <Vhost Name="bar.com">
     <VhostLink Condition="4xx">helloworld.com</VhostLink>
   </Vhost>

   // 원본서버에서 403, 404 또는 5xx로 응답했을 때 해당 요청을 example.com에게 위임한다.
   <Vhost Name="helloworld.com">
     <VhostLink Condition="403,404,5xx">example.com</VhostLink>
   </Vhost>

   // 더 이상 위임하지 않는다.
   <Vhost Name="example.com">
   </Vhost>

.. figure:: img/adv_vhost_link_worst.png
   :align: center

   억지스럽지만 가능하다.

위 예제의 경우 foo.com의 :ref:`admin-log-access` 는 다음과 같다. ::

   #Fields: date time s-ip cs-method cs-uri-stem ...(중략)... vhostlink
   2016.11.24 16:52:24 220.134.10.5 GET /test.jpg ...(중략)... bar.com+helloworld.com+example.com

다음의 경우 링크는 즉시 중단된다.

* 대상 가상호스트가 존재하지 않는 경우 (foo.com -> ?)
* 자기 자신을 대상 가상호스트로 지정한 경우 (foo.com -> foo.com)
* 재귀링크(Recursive Link)가 발생한 경우 (foo.com -> bar.com -> foo.com)
