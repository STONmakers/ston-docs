.. _getting-started:

시작하기
******************

이 장에서는 STON을 설치한 뒤 샘플 가상호스트를 구성하여 실행하는 것까지의 내용을 다룬다.

.. toctree::
   :maxdepth: 2


시스템 구성과 설치
====================================
STON은 표준 Linux 서버에서 동작하도록 개발되었다. 개발 단계부터 HW뿐만 아니라 
특정 OS버전, 파일시스템 등 종속성을 가질 요소는 최대한 배제하였다. 고객이 합리적인 
장비를 선택할 수 있도록 돕는 것은 매우 중요하다. 왜냐하면 서비스의 특성과 규모에 
따라 적절한 서버를 구성하는 것부터 서비스의 시작이기 때문이다.


서버 구성
-----------------------------------------------

일반적인 서버 선택의 기준은 CPU, 메모리, 디스크 이다. 10Gbps급의 높은 퍼포먼스를
요구하는 서비스의 경우 각 구성요소가 서비스 특성을 만족해야 기대 성능을 얻을 수 있다.

-  **CPU**

   Quad 코어 이상을 권장한다. STON은 Many-Core에 대해 확장성(Scalability)를 
   가진다. 코어가 많으면 많을수록 초당 처리량은 증가한다. 단, 높은 처리량이 반드시 
   높은 트래픽을 의미하는 것은 아니다.

   .. figure:: img/10g_cpu.jpg
      :align: center

      클라이언트가 많을수록 많은 CPU는 힘이 된다.
    
   평균파일 크기가 4KB인 서비스는 약 26만번을 전송해야 1GB파일을 
   한번 전송한 것과 같은 대역폭을 사용한다. CPU선택의 가장 큰 기준은 얼마나 
   많은 동시접속을 처리하는가에 있다.
   

-  **메모리**

   4GB이상을 권장한다. 자주 접근되는 콘텐츠를 메모리에 상주시켜야 
   성능이 향상되는데, 물리적인 메모리가 부족하면 아무래도 디스크 부하가 증가할 수 밖에 없다.
   서비스되는 콘텐츠 개수와 상관없이 콘텐츠 크기가 커서 디스크 I/O 부하가 높다면 
   메모리를 증설하여 부하를 낮출 수 있다.


-  **디스크**

   OS를 포함하여 최소 3개 이상을 권장한다. 디스크 역시 많으면 많을수록 많은 
   콘텐츠를 캐싱할 수 있을 뿐만 아니라 I/O부하를 분산할 수 있기 때문이다.
   
   .. figure:: img/02_disk.png
      :align: center
      
      항상 OS와 STON은 별도의 디스크로 구성한다.
   
   일반적으로 OS가 설치된 디스크에 STON을 설치한다. 로그 역시 같은 디스크에
   구성하는 것이 일반적이다. 로그는 서비스 상황을 실시간으로 기록하기 
   때문에 항상 Write 부하를 발생시킨다. 
   
   STON은 디스크를 RAID 0처럼 사용한다. 퍼포먼스와 RAID의 상관여부는 
   고객 서비스 특성에 따라 달라지므로 딱 잘라 말할 수 없다. 하지만 파일 변경이 
   빈번하지 않고 컨텐츠의 크기가 메모리 캐싱크기보다 훨씬 큰 경우 RAID를 통한 
   Read속도 향상이 도움이 될 수 있다.



OS 구성
-----------------------------------------------

가장 기본적인 형태로 설치한다. 표준 64bit Linux 배포판(Cent 6.2이상, Ubuntu 10.04이상)
이라면 정상동작한다. 패키지에 의존성을 가지지 않는다.



설치
-----------------------------------------------

1. 최신버전의 STON을 다운로드 받는다. ::

      [root@localhost ~]# wget  http://foobar.com/ston/ston.2.0.0.rhel.2.6.32.x64.tar.gz
      --2014-06-17 13:29:14--  http://foobar.com/ston/ston.2.0.0.rhel.2.6.32.x64.tar.gz
      Resolving foobar.com... 192.168.0.14
      Connecting to foobar.com|192.168.0.14|:80... connected.
      HTTP request sent, awaiting response... 200 OK
      Length: 71340645 (68M) [application/x-gzip]
      Saving to: “ston.2.0.0.rhel.2.6.32.x64.tar.gz”
      
      100%[===============================================>] 71,340,645  42.9M/s   in 1.6s
      
      2014-06-17 13:29:15 (42.9 MB/s) - “ston.2.0.0.rhel.2.6.32.x64.tar.gz” saved [71340645/71340645]


2. 압축을 해지한다. ::

		[root@localhost ~]# tar -zxf ston.2.0.0.rhel.2.6.32.x64.tar.gz

3. 설치 스크립트를 실행한다. ::

		[root@localhost ~]# ./ston.2.0.0.rhel.2.6.32.x64.sh

4. 설치과정은 install.log에 기록된다. 로그를 통해 설치 중 발생하는 문제를 알 수 있다. ::

      #DownloadURL: http://foobar.com/ston/ston.2.0.0.rhel.2.6.32.x64.tar.gz
      #DownloadTime: 13 sec
      #Target: STON 2.0.0
      #Date: 2014.03.03 16:48:35
      Prepare for STON 2.0.0 install process
          Stopping STON...
              STON stopped
      [Copying files]
          `./fuse.conf' -> `/etc/fuse.conf'
          `./libfuse.so.2' -> `/usr/local/ston/libfuse.so.2'
          `./libtbbmalloc_proxy.so' -> `/usr/local/ston/libtbbmalloc_proxy.so'
          `./start-stop-daemon' -> `/usr/sbin/start-stop-daemon'
          `./libtbbmalloc_proxy.so.2' -> `/usr/local/ston/libtbbmalloc_proxy.so.2'
          `./libtbbmalloc.so' -> `/usr/local/ston/libtbbmalloc.so'
          `./libtbbmalloc.so.2' -> `/usr/local/ston/libtbbmalloc.so.2'
          `./libtbb.so' -> `/usr/local/ston/libtbb.so'
          `./libtbb.so.2' -> `/usr/local/ston/libtbb.so.2'
          `./stond' -> `/usr/local/ston/stond'
          `./stonx' -> `/usr/local/ston/stonx'
          `./stonr' -> `/usr/local/ston/stonr'
          `./stonu' -> `/usr/local/ston/stonu'
          `./stonapi' -> `/usr/local/ston/stonapi'
          `./server.xml.default' -> `/usr/local/ston/server.xml.default'
          `./vhosts.xml.default' -> `/usr/local/ston/vhosts.xml.default'
          `./ston_format.sh' -> `/usr/local/ston/ston_format.sh'
          `./ston_diskinfo.sh' -> `/usr/local/ston/ston_diskinfo.sh'
          `./wm.sh' -> `/usr/local/ston/wm.sh'
      [Exporting config files]
          #Export so directory
          /usr/local/ston/ to ld.so.conf
          #Export sysctl to /etc/sysctl.conf
          vm.swappiness=0
          vm.min_free_kbytes=524288
          #Export sudoers for WM
          Defaults    !requiretty
          winesoft ALL=NOPASSWD: /etc/init.d/ston stop, /etc/init.d/ston start, /bin/ps -ef
      [Configuring STON daemon script]
          STON deamon activate in run-level 2345.
      [Installing sub-packages]
          curl installed.
          libjpeg installed.
          libgomp installed.
          rrdtool installed.
      [Installing WM]
          Stopping WM...
          WM stopped
          `./wm.server_default.xml' -> `/usr/local/ston/wm/tmp/conf/server_default.xml'
          `./wm.vhost_default.xml' -> `/usr/local/ston/wm/tmp/conf/vhost_default.xml'
          WM configuration found. Current WM port : 8500
          PHP module for Legacy(CentOS 5.5) installed
          `./libphp5.so.5.5' -> `/usr/local/ston/wm/modules/libphp5.so'
          WM installation almost complete. Changing WM privileges.
      Installation successfully complete


업데이트
-----------------------------------------------
최신버전이 배포되면 stonu명령어로 업데이트할 수 있다. ::

	./stonu 2.0.1
	
또는 WM(Web Management)을 통해 업데이트를 진행할 수 있다.

   .. figure:: img/conf_update1.png
      :align: center


실행하기
====================================

기본 설치경로는 /usr/local/ston/ 이다. 최초 설치시 필수 설정파일인 server.xml과 
vshost.xml이 존재하지 않는다. 설치경로의 server.xml.default와 vhosts.xml.default를 
복사 또는 수정하여 설정하길 권장한다. *.default파일은 항상 최신패키지와 함께 배포된다.


샘플 가상호스트 생성
-----------------------------------------------
vhost.xml 파일을 열어 다음과 같이 편집한다. ::

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>welcome.winesoft.co.kr</Address>
            </Origin>
        </Vhost>
    </Vhosts>   


STON 실행
-----------------------------------------------
1. 발급받은 license.xml을 설치 경로에 복사한다.

2. server.xml을 열어 <Storage>를 구성한다. ::

      <Server>
          <Cache>
              <Storage>
                  <Disk>/cache1/</Disk>
                  <Disk>/cache2/</Disk>
              </Storage>
          </Cache>
      </Server>
      
.. note::

   STON은 기본적으로 디스크를 저장공간으로 사용하기 때문에, 디스크가 설정되어 있지 않으면
   구동되지 않는다. 자세한 내용은 다음 장에서 설명한다.

3. STON을 실행한다.  ::

		[root@localhost ~]# service ston start

    STON을 중지하고 싶다면 stop 명령을 사용한다.  ::

		[root@localhost ~]# service ston stop


가상호스트 동작확인
-----------------------------------------------
(Windows 7 기준) C:\\Windows\\System32\\drivers\\etc\\hosts\\hosts 파일에 다음과 같이
www.example.com 도메인을 설정한다. ::

    192.168.0.100        www.example.com

브라우저로 www.example.com에 접근했을 때 다음 페이지가 정상적으로 서비스되면 성공입니다.

   .. figure:: img/welcome_page.png
      :align: center


설정 Reload
-----------------------------------------------
설정파일을 변경한 뒤에는 반드시 Reload를 호출해야 설정이 반영된다. ::

    ./stonapi conf/reload

.. note::

   어떠한 경우에도 STON은 설정파일을 수정하지 않는다. 관리자는 서비스 반영을 원하는 
   시점에 분명하게 reload를 수행하여 설정을 반영해야 한다.
