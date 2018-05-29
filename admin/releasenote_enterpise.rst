.. _release_enterprise:

Appendix E: 릴리스 노트  ``[Enterprise]``
***********************

v18.x
====================================

18.05.1 (2018.5.29)
----------------------------

**기능개선/정책변경**

- :ref:`media-hls` - 키프레임의 간격이 불규칙한 영상에 대한 호환성 강화

.. warning::

   이전 버전과 :ref:`media-hls` 의 MPEG2-TS가 호환되지 않습니다.


**버그수정**

 -  :ref:`handling_http_requests_header_lastmodifiedcheck` - ``orlater`` 로 설정 할 경우 최초 캐싱 시 304 응답을 할 수 있는 문제 수정


18.05.0 (2018.5.15)
----------------------------

-  클라이언트 요청 :ref:`handling_http_requests_header_if_range` 헤더 지원 
-  원본 요청 시 :ref:`origin_header_if_range` 헤더 지원
-  :ref:`handling_http_requests_header_lastmodifiedcheck` 설정기능 추가
-  :ref:`bypass-put` 기능 추가



18.04.0 (2018.4.26)
----------------------------

**기능개선/정책변경**

- :ref:`media-dims` - :ref:`media-dims-annotation` 기능 추가


.. note::

   v2.5.13 이후부터 새로운 Versioning으로 제공됩니다.

   -  ``CDN`` - v2.5.14와 같은 기존 Versioning
   -  ``Enterprise`` - v.18.04.0과 같은 연도.월 형태의 새로운 Versioning
