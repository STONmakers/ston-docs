.. _release:

Appendix B: Release Notes
***********************


v2.0.x
====================================

2.0.4 (FEB 27, 2015)
----------------------------

**기능개선/Policy Update**

   - :ref:`origin-balancemode` 의 ``Hash`` 알고리즘 변경
   
     | **Before.** hash(URL) / 서버대수
     | **After.** `Consistent Hashing <http://en.wikipedia.org/wiki/Consistent_hashing>`_
     |     
   - :ref:`access-control-vhost` 를 통해 Redirect 할 때 클라이언트가 요청한 URI을 파라미터로 입력할 수 있다.
   
**Bug Fixes**

   - 캐싱된 파일이 삭제되지 않아 디스크가 꽉 차던 증상
   
   

2.0.3 (2015.2.9)
----------------------------

**기능개선/정책변경**

   - DIMS 내재화 및 고도화
   - WM - 트래픽 관련 안내 메세지 추가
   
**버그 수정**

   - WM - 신규 가상호스트 생성이 실패 하는 버그 수정


2.0.2 (2015.1.28)
----------------------------

- 원본서버에 캐싱요청할 때 클라이언트가 보낸 User-Agent헤더 값을 보낼 수 있다.

**버그 수정**

   - MDAT 길이가 1인 MP4파일의 Trimming이 되지 않던 증상
   - WM - 클러스터 내의 다른 서버 그래프가 표시되지 않던 증상
   - WM - 클러스터 내의 다른 서버들이 현재 서버로 보여지던 증상


2.0.1 (2014.12.30)
----------------------------

**버그 수정**

   - HitRatio그래프가 0으로 표시되던 증상


2.0.0 (2014.12.17)
----------------------------

- 원본에서 다운로드된 크기만큼만 디스크 공간사용. ( :ref:`origin_partsize` 참조)
- :ref:`env-cache-resource` 기능추가.
- TLS 1.1 지원.
- AES-NI를 통해 :ref:`https-aes-ni` 지원.
- ECDHE 계열의 CipherSuite를 지원. ( :ref:`https-ciphersuite` 참조)
- :ref:`admin-log-dns` 추가
- 원본서버가 Domain일 경우 각 IP별 TTL을 사용하도록 정책변경.
- 원본 :ref:`origin_exclusion_and_recovery` 추가
- 원본 :ref:`origin-health-checker` 추가
- :ref:`adv_topics_sys_free_mem` 추가
- 기타

  - 최소 실행환경 변경. (Cent 6.2이상, Ubuntu 10.01 이상)
  - 설치 패키지에 NSCD데몬이 탑재.
  - :ref:`media-dims` 기본 탑재.
  - :ref:`getting-started-reset` 후 STON 재시작하도록 변경.
  - ``<DNSBackup>`` 기능 삭제
  - ``<MaxFileCount>`` 기능 삭제.
  - ``<Distribution>`` 기능 삭제. :ref:`origin-balancemode` 기능에 통합.

