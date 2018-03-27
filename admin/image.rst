.. _media-dims:

17장. 이미지/DIMS
******************

.. note::

   - `[동영상 강좌] 해보자! STON Edge Server - Chapter 4. 실시간 이미지 가공 <https://youtu.be/Pdfe-HbtXVs?list=PLqvIfHb2IlKeZ-Eym_UPsp6hbpeF-a2gE>`_

이 장에서는 이미지를 전송시점에 on-the-fly로 변환/전송하는 DIMS(딤스)에 대해 다룬다.
DIMS(Dynamic Image Management System)는 원본이미지를 다양한 형태로 가공하는 기능이다.

.. figure:: img/dims.png
   :align: center

   다양한 동적 이미지 가공

이미지는 동적으로 생성되며 원본 이미지 URL뒤에 약속된 키워드와 가공옵션을 붙여서 호출한다.
가공된 이미지는 캐싱되어 원본서버 이미지가 바뀌지 않는 이상 다시 가공되지 않는다.

예를 들어 원본 파일이 /img.jpg라면 다음과 같은 형식으로 이미지를 가공할 수 있다.
("12AB"는 약속된 Keyword이다.) ::

   http://image.example.com/img.jpg    // 원본 이미지
   http://image.example.com/img.jpg/12AB/optimize
   http://image.example.com/img.jpg/12AB/resize/500x500/
   http://image.example.com/img.jpg/12AB/crop/400x400/
   http://image.example.com/img.jpg/12AB/composite/watermark1/

``<Dims>`` 는 별도로 설정하지 않으면 모두 비활성화되어 있다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <Dims Status="Active" Keyword="dims" MaxSourceSize="10" OnFailure="message" />

-  ``<Dims>``

   - ``Status`` DIMS활성화 ( ``Active`` 또는 ``Inactive`` )
   - ``Keyword`` 원본과 DIMS를 구분하는 키워드
   - ``MaxSourceSize (기본: 10MB)`` 변환을 허용할 최대 원본 이미지 크기 (단위: MB)
   - ``OnFailure`` 이미지 변환실패 시 동작방식

     - ``message (기본)`` 500 Internal Error로 응답한다. 본문에는 구체적인 실패 이유를 명시한다.

       - ``The original file was not successfully downloaded.`` 원본이미지를 완전하게 다운로드 하지 못했다.
       - ``The original file size is too large.`` 원본이미지 크기가 ``MaxSourceSize`` 를 넘어 변환하지 못했다.
       - ``The original file loading failed.`` 원본 이미지 데이터를 불러오지 못했다.
       - ``Image converting failed or invalid DIMS command.`` 잘못된 명령어 또는 지원되지 않는 이미지등으로 인해 변환하지 못했다.

     - ``redirect`` 원본 이미지 주소로 302 Redirect한다.



.. toctree::
   :maxdepth: 2


최적화
====================================

최적화란 이미지 품질을 저하시키지 않으면서 이미지를 압축하는 과정이다.
JPEG, JPEG-2000, Loseless-JPEG 이미지만 지원이 가능하다.
이미 다른 도구등을 통해 최적화된 이미지는 더 이상 최적화되지 않는다. ::

   http://image.example.com/img.jpg/dims/optimize

최적화는 키워드 이외 별도의 옵션을 가지지 않는다.
그러므로 다른 변환조건과 조합할 때 맨 뒤에 명시하는 편이 바람직하다. ::

   http://image.example.com/img.jpg/dims/resize/100x100/optimize

다른 모든 DIMS기능이 시스템 자원을 많이 사용하지만 그 중에서도 최적화가 가장 무거운 작업이다.
다음은 HitRatio가 0%인 상태에서 이미지 크기별 성능 테스트 결과이다.

-  ``OS`` CentOS 6.2 (Linux version 2.6.32-220.el6.x86_64 (mockbuild@c6b18n3.bsys.dev.centos.org) (gcc version 4.4.6 20110731 (Red Hat 4.4.6-3) (GCC) ) #1 SMP Tue Dec 6 19:48:22 GMT 2011)
-  ``CPU`` `Intel(R) Xeon(R) CPU E3-1230 v3 @ 3.30GHz (8 processors) <http://www.cpubenchmark.net/cpu.php?cpu=Intel+Xeon+E3-1230+v3+%40+3.30GHz>`_
-  ``RAM`` 16GB
-  ``HDD`` SMC2108 SAS 275GB X 3EA

====== ======= ============= ======================= ================== ================
크기   처리량  응답속도(ms)  클라이언트 트래픽(Mbps) 원본 트래픽(Mbps)  트래픽 절감률(%)
====== ======= ============= ======================= ================== ================
16KB   720     19.32         46.32                   92.62              49.99
32KB   680     20.68         86.42                   165.08             47.65
64KB   285     50.16         80.67                   150.96             46.56
128KB  274     57.80         164.35                  276.52             40.56
256KB  210     80.74         99.42                   432.35             77.00
512KB  113     156.18        160.54                  436.04             63.18
1MB    20      981.07        90.62                   179.88             49.62
====== ======= ============= ======================= ================== ================

약 50%내외의 트래픽 절감률이 있으므로 매우 효과적이다.
다시 한번 말하지만 최적화는 매우 무거운 작업이다.
표를 통해 알 수 있듯이 이미지 크기가 가장 큰 변수가 된다.

때문에 충분한 고려없이 서비스에 적용했다가는 큰 낭패를 볼 수 있다.
적당한 :ref:`adv_topics_req_hit_ratio` 가 있는 상황이 바람직하나,
그렇지 않다면 서비스 규모에 맞게 물리적인 CPU자원을 충분히 확보해야 한다.


.. note::

   메타정보만을 삭제하고 싶을 경우 아래 명령어를 이용한다. ::

      http://image.example.com/img.jpg/dims/strip/true



잘라내기
====================================

좌상단을 기준으로 원하는 영역만큼 이미지를 잘라낸다.
영역은 **width x height{+-}x{+-}y{%}** 로 표현한다.
다음은 좌상단 x=20, y=30을 기준으로 width=100, height=200만큼 잘라내는 예제다. ::

   http://image.example.com/img.jpg/dims/crop/100x200+20+30/


이미지 중앙을 기준으로 하고 싶은 경우 cropcenter명령어를 사용한다. ::

   http://image.example.com/img.jpg/dims/cropcenter/100x200+20+30/



Thumbnail 생성
====================================

Thumbnail 을 생성한다.
크기와 옵션은 **width x height{%} {@} {!} {<} {>}** 로 표현한다.
기본적으로 이미지의 가로와 세로는 최대값을 사용한다.
이미지를 확대 또는 축소하여도 가로 세로 비율은 유지된다.
정확하게 지정한 크기로 이미지를 조절할 때는 크기 뒤에 느낌표(!)를 추가한다.
**640X480!** 라는 표현은 정확하게 640x480 크기의 Thumbnail을 생성한다는 뜻이다.
만약 가로 또는 세로 크기만 지정된 경우, 생략된 값은 가로/세로 비율에 의해 자동결정 된다.

예를 들어 **/thumbnail/100/** 은 가로 크기에 맞추어 세로 크기가 결정되며
**/thumbnail/x200/** 은 세로 크기에 맞추어 가로 크기가 결정된다.
가로/세로 크기를 이미지의 크기에 맞추어 백분율(%)로 표현할 수 있다.
이미지 크기를 늘리려면, 100 보다 큰 값(예 : 125 %)을 사용한다.
이미지 크기를 줄이려면 100 미만의 비율을 사용한다.
URL Encoding규칙에 따라 %문자가 %25로 인코딩 됨을 명심해야 한다.

예를 들어 50%라는 표현은 50%25로 인코딩 된다.
다음은 width=78, height=110크기의 Thumbnail을 생성하는 예제다. ::

   http://image.example.com/img.jpg/dims/thumbnail/78x110/


Resizing
====================================

이미지 크기를 변경한다.
크기는 **width x height** 로 표현한다.
이미지는 변경되어도 비율은 유지된다.
다음은 원본 이미지를 width=200, height=200크기로 변경하는 예제다. ::

   http://image.example.com/img.jpg/dims/resize/200x200/

그 외 명령어는 다음과 같다.

-  **resizec** - 축소하면 resize와 동일하지만, 확대하면 이미지는 유지되고 캔버스 크기만 확대된다.
-  **extent** - 캔버스만 조절하는 명령어. 축소하면 crop과 동일한 효과를 내지만, 확대하면 resizec와 동일하게 확대된다.
-  **trim** - 상하좌우 흰색배경을 제거한다.



Format 변경
====================================

이미지 포맷을 변경한다.
지원되는 포맷은 "png", "jpg", "gif" 이다.
다음은 JPG를 PNG로 변환하는 예제다. ::

   http://image.example.com/img.jpg/dims/format/png/


품질 변경
====================================

이미지 품질을 조절한다.
이 기능은 전송되는 이미지 용량을 줄일 수 있어서 효과적이다.
유효 범위는 0부터 100까지다.
다음은 이미지 품질을 25%로 조절하는 예제다. ::

   http://image.example.com/img.jpg/dims/quality/25/


이펙트
====================================

이미지에 다양한 이펙트를 줄 수 있다.

================ ===================== =================
설명              명령어                  변수
================ ===================== =================
반전               invert                true 또는 false
그레이 스케일        grayscale            true 또는 false
대칭이동            flipflop             vertical
밝기조절            bright                0 ~ 100
회전               rotate                0 ~ 360 (도)
세피아              sepia                 0 ~ 1
모서리 라운드        round                 0 ~ 90
================ ===================== =================



합성
====================================

두 이미지를 합성한다.
앞서 설명한 기능과는 다르게 합성조건은 미리 설정되어 있어야 한다.
주로 워터마크 효과를 내기 위해 사용된다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <Dims Status="Active" Keyword="dims" port="8500">
      <Composite Name="water1" File="/img/small.jpg" />
      <Composite Name="water2" File="/img/medium.jpg" Gravity="se" Geometry="+0+0" Dissolve="50" />
      <Composite Name="water_ratio" File="/img/wmark_s.png" Gravity="s" Geometry="+0+15%" Dissolve="100" />
   </Dims>

-  ``<Composite>``

    이미지 합성조건을 설정한다. 속성에 의해 정해지며 별도의 값을 가지지 않는다.

    -  ``Name`` 호출될 이름을 지정한다.
       '/'문자는 입력할 수 없다.
       URL의 "/composite/" 뒤에 위치한다.

    -  ``File`` 합성할 이미지파일 경로를 지정한다.

    -  ``Gravity (기본: c)`` 합성할 위치는 좌측상단부터 9가지의 포인트(nw, n, ne, w, c, e, sw, s, se)가 존재한다.

       .. figure:: img/conf_dims2.png
          :align: center

          Gavity 기준점

    -  ``Geometry (기본: +0+0)`` ``Gravity`` 기준으로 합성할 이미지 위치를 설정한다.
       {+-}x{+-}y. 붉은색 원은 Gravity속성에 따라 +0+0이 의미하는 기준점으로 +x+y의
       값이 커질수록 이미지 안쪽으로 배치된다.
       초록색 화살표는 +x, 보라색 화살표는 +y가 증가하는 방향이다.
       -x-y를 사용하면 대상 이미지의 바깥에 위치하게 되어 결과 이미지에서는 보여지지 않는다.
       이 속성은 다소 복잡해 보이지만 이미지 크기를 자동으로 계산하여 배치하므로
       일관된 결과물을 얻을 수 있어서 효과적이다.
       또한 +x%+y% 처럼 %옵션을 주어 비율로 배치할 수도 있다.

    -  ``Dissolve (기본: 50)`` 합성할 이미지의 투명도(0~100).

``<Composite>`` 을 설정했다면 ``Name`` 속성을 사용하여 이미지를 합성할 수 있다. ::

    http://image.example.com/img.jpg/dims/composite/water1/



원본이미지 조건판단
====================================

원본 이미지 조건에 따라 동적으로 가공 옵션을 다르게 적용할 수 있다.
예를 들어 1024 X 768 이하의 이미지는 품질을 50%로 떨어트리고 그 이상의
이미지는 1024 X 768로 크기변환을 하려면 다음과 같이 ``<ByOriginal>`` 을 설정한다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <Dims Status="Active" Keyword="dims" port="8500">
      <ByOriginal Name="size1">
         <Condition Width="1024" Height="768">/quality/50/</Condition>
         <Condition>/resize/1024x768/</Condition>
      </ByOriginal>
   </Dims>

-  ``<ByOriginal>``
   ``Name`` 속성으로 호출한다.
   하위에 다양한 조건의 ``<Condition>`` 을 설정한다.

-  ``<Condition>``
   조건에 만족하는 경우 설정된 변환을 수행한다.

   - ``Width`` 가로 길이가 설정 값보다 작으면 적용된다.
   - ``Height`` 세로 길이가 설정 값보다 작으면 적용된다.

   조건을 설정하지 않으면 원본 이미지 크기에 상관없이 변환된다.

``<Condition>`` 은 명시된 순서대로 적용된다.
그러므로 작은 이미지 조건을 먼저 배치해야 한다.
다음과 같이 호출한다. ::

   http://image.example.com/img.jpg/dims/byoriginal/size1/

또 다른 예로 이미지 크기에 따라 다른 ``<Composite>`` 조건을 줄 수 있다.
이런 경우 다음과 같이 사전에 정의된 ``<Composite>`` 의 ``Name`` 으로 설정한다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <Dims Status="Active" Keyword="dims" port="8500">
      <Composite Name="water1" File="/img/small.jpg" />
      <Composite Name="water2" File="/img/medium.jpg" Gravity="se" Geometry="+0+0" Dissolve="50" />
      <Composite Name="water3" File="/img/big.jpg" Gravity="se" Geometry="+10+10" Dissolve="50" />
      <ByOriginal Name="size_water">
         <Condition Width="400">/composite/water1/</Condition>
         <Condition Width="800">/composite/water2/</Condition>
         <Condition>/composite/water3/</Condition>
      </ByOriginal>
   </Dims>

다음과 같이 호출하면 원본 이미지 크기에 따라 합성이 적용된다. ::

   http://image.example.com/img.jpg/dims/byoriginal/size_water/


.. _media-dims-anigif:

Animated GIF
====================================

Animated GIF에 대해서도 모든 DIMS변환이 동일하게 적용된다.
처리 순서는 다음과 같다.

1. Animated GIF를 낱개의 이미지들로 분해한다.
2. 각각의 이미지를 변환한다.
3. 변환된 이미지를 Animated GIF로 결합한다.

결합된 이미지가 많을수록 처리비용이 높아 서비스 품질이 저하될 수 있다.
이런 경우 첫 번째 이미지에 대해서만 변환하도록 설정하면 처리 비용을 낮출 수 있다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <Dims FirstFrameOnly="OFF" />

-  ``FirstFrameOnly (기본: OFF)`` ON인 경우 Animated GIF의 첫 장면만 변환한다.

다음과 같이 URL을 호출할 때 ``FirstFrameOnly`` 옵션을 명시적으로 지정할 수 있다. ::

   http://image.example.com/img.jpg/dims/firstframeonly/on/resize/200x200/
   http://image.example.com/img.jpg/dims/firstframeonly/off/resize/200x200/

위와 같이 URL에 명시적으로 지정되어 있는 경우 설정보다 우선한다.


.. note::

   ``limit`` 명령어를 통해 Animated GIF의 프레임 수를 조절할 수 있다. ::
      
      http://image.example.com/img.jpg/dims/limit/3
      http://image.example.com/img.jpg/dims/limit/3/resize/200x200



기타
====================================

이상의 기본기능을 결합하여 복합적인 이미지 가공을 할 수 있다.
예를 들어 Thumbnail생성(78x110), 포맷을 JPG에서 PNG로 변환, 품질 50% 이상의 옵션을 한번의 호출로 실행할 수 있다. ::

   http://image.example.com/img.jpg/dims/thumbnail/78x110/format/png/quality/50/

DIMS는 URL을 이용하여 이미지 가공이 이루어진다.
그러므로 URL에 영향을 주는 다른 옵션들 때문에 원하지 않는 결과가 얻어지지 않도록 주의해야 한다.

-  :ref:`caching-policy-applyquerystring` 이 ``OFF`` 라면 키워드 이전의 QueryString이 무시된다. ::

      http://image.example.com/img.jpg?session=5234&type=37/dims/resize/200x200/

   위와 같은 호출에 이 설정이 ``ON`` 이라면 입력된 URL 그대로 인식되지만 OFF라면 다음과 같이 인식된다. ::

      http://image.example.com/img.jpg/dims/resize/200x200/

-  :ref:`caching-policy-casesensitive` 이 ``OFF`` 라면 모든 URL을 소문자로 변환하여 처리한다.
   그러므로 DIMS 키워드에 대문자가 포함되었다면 키워드를 인식하지 못한다.
   항상 키워드는 소문자로 사용하는 것이 좋다.
