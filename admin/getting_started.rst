.. _getting-started:

2장. 시작하기
******************

이장에서는 시스템 구성으로부터 설치 그리고 예제 가상호스트까지 구성해본다. 
텍스트 편집기만 있으면 누구나 할 수 있다.

STON은 표준 Linux 서버에서 동작하도록 개발되었다. 
개발 단계부터 HW뿐만 아니라 OS, 파일시스템 등 종속성을 가질 수 있는 요소는 최대한 배제하였다. 
고객이 합리적인 장비를 선택할 수 있도록 돕는 것은 매우 중요하다. 
왜냐하면 서비스의 특성과 규모에 따라 적절한 서버를 구성하는 것이 서비스의 시작이기 때문이다.


.. toctree::
   :maxdepth: 2
   

.. _getting-started-serverconf:

서버 구성
====================================

일반적으로 서버를 구성할 때는 CPU, 메모리, 디스크를 주로 고려한다.
10Gbps급의 높은 성능를 요구하는 서비스라면 각 구성요소가 서비스 특성을 만족해야 원하는 성능을 얻을 수 있다.

-  **CPU**

   Quad 코어 이상을 권장한다. 
   STON은 Many-Core에 대해 확장성(Scalability)를 가진다. 
   코어가 많으면 많을수록 초당 처리량은 증가한다. 
   단, 높은 처리량이 반드시 높은 트래픽을 의미하는 것은 아니다.

   .. figure:: img/10g_cpu.jpg
      :align: center

      클라이언트가 많을수록 많은 CPU는 힘이 된다.
    
   4KB인 파일을 약 26만번을 전송하는 것과 1GB파일을 한번 전송하는 것은 같은 대역폭을 사용한다. 
   CPU선택의 가장 큰 기준은 얼마나 많은 동시접속을 처리하는가이다.
   

-  **메모리**

   Memory-Indexing 방식을 사용하므로 4GB이상을 권장한다. ( :ref:`adv_topics_mem` 참조)
   자주 접근되는 콘텐츠는 항상 메모리에 상주하지만 그렇지 않은 컨텐츠는 디스크에서 로딩한다.
   따라서 컨텐츠가 많고 집중도가 낮다면(Long-tail) 디스크 부하 증가로 성능이 저하될 수 있다.
   서비스되는 콘텐츠 개수와 상관없이 콘텐츠 크기가 커서 디스크 I/O 부하가 높다면 
   메모리를 증설하여 부하를 낮출 수 있다.


-  **디스크**

   OS를 포함하여 최소 3개 이상을 권장한다. 
   디스크 역시 많으면 많을수록 많은 콘텐츠를 캐싱할 수 있을뿐만 아니라 I/O부하도 분산된다.
   
   .. figure:: img/02_disk.png
      :align: center
      
      항상 OS와 STON은 콘텐츠와 별도의 디스크로 구성한다.
   
   일반적으로 OS가 설치된 디스크에 STON을 설치한다. 
   로그 역시 같은 디스크에 구성하는 것이 일반적이다. 
   로그는 서비스 상황을 실시간으로 기록하기 때문에 항상 Write부하가 발생한다.
   
   STON은 디스크를 RAID 0처럼 사용한다. 
   성능와 RAID의 상관여부는 고객 서비스 특성에 따라 달라진다. 
   하지만 파일 변경이 빈번하지 않고 콘텐츠의 크기가 물리적 메모리 크기보다 
   훨씬 큰 경우 RAID를 통한 Read속도 향상이 효과적일 수 있다.


.. _getting-started-os:

OS 구성
====================================

가장 기본적인 형태로 설치한다. 
표준 64bit Linux 배포판(Cent 6.2이상, Ubuntu 10.04이상) 이라면 정상동작한다. 
패키지 의존성을 가지지 않는다.


.. _getting-started-install:

설치
====================================

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


.. _getting-started-license:

라이선스 발급
====================================

신규 고객의 경우 다음 절차를 통해 라이선스를 발급한다.

* `신청양식 <http://ston.winesoft.co.kr/EULR.doc>`_ 작성
* license@winesoft.co.kr 로 전송
* 확인절차 후 발급

라이선스 파일(license.xml)이 반드시 설치경로에 존재해야 STON이 정상적으로 구동된다.


.. _getting-started-update:

업데이트
====================================
최신버전이 배포되면 stonu명령어로 업데이트할 수 있다. ::

	./stonu 2.0.1
	
또는 :ref:`wm` 의 :ref:`wm-update` 를 통해 간편하게 업데이트를 진행할 수 있다.

   .. figure:: img/conf_update1.png
      :align: center


.. _getting-started-run:

실행하기
====================================

보통 기본경로에 STON을 설치한다. ::

    /usr/local/ston/

다음 파일 중 하나라도 존재하지 않거나 XML문법에 맞지 않을 경우 실행되지 않는다.

- license.xml
- server.xml
- vhosts.xml

최초 설치시 모든 XML파일이 존재하지 않는다. 
배포받은 라이센스파일을 설치 경로에 복사한다.
그리고 설치경로의 server.xml.default와 vhosts.xml.default를 복사 또는 수정하여 설정하길 바란다. 
*.default파일은 항상 최신패키지와 함께 배포된다.


.. _getting-started-samplevhost:

Hello World
====================================
vhosts.xml 파일을 열어 다음과 같이 편집한다. ::

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>hello.winesoft.co.kr</Address>
            </Origin>
        </Vhost>
    </Vhosts>   


.. _getting-started-runston:

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

   STON은 기본적으로 디스크를 저장공간으로 사용하기 때문에 디스크가 설정되어 있지 않으면 구동되지 않는다. 
   자세한 내용은 다음 장에서 설명한다.

3. STON을 실행한다.  ::

      [root@localhost ~]# service ston start

   STON을 중지하고 싶다면 stop 명령을 사용한다.  ::

      [root@localhost ~]# service ston stop


.. _getting-started-runcheck:

가상호스트 동작확인
-----------------------------------------------

(Windows 7 기준) C:\\Windows\\System32\\drivers\\etc\\hosts 파일에 다음과 같이
www.example.com 도메인을 설정한다. ::

    192.168.0.100        www.example.com

브라우저로 www.example.com에 접근했을 때 다음 페이지가 정상적으로 서비스되면 성공입니다.

   .. figure:: img/helloworld3.png
      :align: center


.. _getting-started-rrderr:

WM이 느리거나 그래프가 나오지 않는 문제
-----------------------------------------------

설치과정 중 RRD그래프는 동적으로 다운로드 받아서 설치된다. 
제한된 네트워크에서 STON을 설치할 경우 RRD가 제대로 설치되지 않을 수 있다. 
이로 인해 :ref:`wm` 이 매우 느리게 동작하거나 :ref:`api-graph` 가 동작하지 않게 된다.
다음과 같이 수정한다.


**1. rrdtool 설치확인**

   다음과 같이 설치여부를 확인한다. ::
   
      [root@localhost ston]# yum install rrdtool
      Loaded plugins: fastestmirror, security
      Loading mirror speeds from cached hostfile
      * base: centos.mirror.cdnetworks.com
      * elrepo: ftp.ne.jp
      * epel: mirror.premi.st
      * extras: centos.mirror.cdnetworks.com
      * updates: centos.mirror.cdnetworks.com
      Setting up Install Process
      Package rrdtool-1.3.8-6.el6.x86_64 already installed and latest version
      Nothing to do
      
   다음은 Ubuntu계열의 경우이다. ::

      root@ubuntu:~# apt-get install rrdtool
      Reading package lists... Done
      Building dependency tree
      Reading state information... Done
      rrdtool is already the newest version.
      The following packages were automatically installed and are no longer required:
        libgraphicsmagick3 libgraphicsmagick++3 libgraphicsmagick1-dev libgraphics-magick-perl libgraphicsmagick++1-dev
      Use 'apt-get autoremove' to remove them.
      0 upgraded, 0 newly installed, 0 to remove and 102 not upgraded.
      
      
**2. RRD 수동설치**

   만약 rrdtool이 yum을 이용해서 설치가 되지 않는다면, 
   OS 버전에 맞는 패키지를 `다운로드 <http://pkgs.repoforge.org/rrdtool/>`_ 후 수동으로 설치한다.   
   
======================================== =================== ======= ============================
Name                                     Last Modified       Size    Description
======================================== =================== ======= ============================
tcl-rrdtool-1.4.7-1.el5.rf.i386.rpm      06-Apr-2012 16:57   36K     RHEL5 and CentOS-5 x86 32bit
tcl-rrdtool-1.4.7-1.el5.rf.x86_64.rpm	 06-Apr-2012 16:57   37K     RHEL5 and CentOS-5 x86 64bit
tcl-rrdtool-1.4.7-1.el6.rfx.i686.rpm     06-Apr-2012 16:57   35K     RHEL6 and CentOS-6 x86 32bit
tcl-rrdtool-1.4.7-1.el6.rfx.x86_64.rpm	 06-Apr-2012 16:57   35K     RHEL6 and CentOS-6 x86 64bit
======================================== =================== ======= ============================


.. _env-vhost-activeorigin:

원본서버
============================================

가상호스트는 원본서버를 대신해 콘텐츠를 서비스하는 것이 목적이다.
서비스 형태에 맞게 다양한 원본서버는 다양하게 접근이 가능하다. ::

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>1.1.1.1</Address>
                <Address>1.1.1.2</Address>
            </Origin>
        </Vhost>
    </Vhosts>

-  ``<Address>``
   가상호스트가 콘텐츠를 복제 할 원본서버 주소.
   개수제한은 없다.
   주소가 2개 이상일 경우 Active/Active방식(Round-Robin)으로 선택된다. 
   원본서버 주소 포트가 80인 경우 생략할 수 있다. 

예를 들어 다른 포트(8080)로 서비스되는 경우 1.1.1.1:8080과 같이 포트번호를 
명시해야 한다. 주소는 {IP|Domain}{Port}{Path}형식으로 8가지 형식이 가능하다.

============================== ==========================
Address                        Host헤더
============================== ==========================
1.1.1.1	                       가상호스트명
1.1.1.1:8080	               가상호스트명:8080       
1.1.1.1/account/dir	           가상호스트명            
1.1.1.1:8080/account/dir       가상호스트명:8080       
example.com	                   example.com             
example.com:8080	           example.com:8080        
example.com/account/dir	       example.com             
example.com:8080/account/dir   example.com:8080
============================== ==========================

:ref:`origin-httprequest` 중 Host헤더를 별도로 설정하지 않는한 표의 Host헤더를 전송한다. ::

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>origin.com:8888/account/dir</Address>            
            </Origin>
        </Vhost>
    </Vhosts>

예를 들어 위와같이 설정하면 원본으로 다음과 같이 요청한다. ::

   GET / HTTP/1.1
   Host: origin.com:8888

.. note:

   원본서버에 example.com/account/dir처럼 경로가 붙어있다면 요청된 URL은 원본서버 주소 경로 뒤에 붙는다. 
   클라이언트가 /img.jpg를 요청하면 최종 주소는 example.com/account/dir/img.jpg가 된다.


.. _env-vhost-standbyorigin:

보조 원본주소
------------------------------------------------

보조 원본서버를 설정한다.::

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>1.1.1.1</Address>
                <Address>1.1.1.2</Address>
                <Address2>1.1.1.3</Address2>
                <Address2>1.1.1.4</Address2>
            </Origin>
        </Vhost>
    </Vhosts>
    
-  ``<Address2>``

   모든 ``<Address>`` 가 정상동작하고 있다면 ``<Address2>`` 는 서비스에 투입되지 않는다. 
   Active서버에 장애가 감지되면 해당 서버를 대체하기 위해 투입되며 
   Active서버가 복구되면 다시 Standby상태로 돌아간다. 
   만약 Standby서버에 장애가 감지되면 해당 Standby서버가 복구되기 전까지 서비스에 투입되지 않는다.


.. _api-etc-help:

API 호출
====================================

HTTP기반의 API를 제공한다.
API 호출권한은 :ref:`env-host` 의 영향을 받는다. 
허가되지 않았다면 곧바로 연결을 종료한다.

STON버전을 확인한다. ::

   http://127.0.0.1:10040/version
    
같은 API를 Linux Shell에서 명령어로 수행한다. ::

   ./stonapi version

.. note:

   HTTP API는 &를 QueryString의 구분자로 인식하지만 Linux 콘솔에서는 다른 의미를 가진다. 
   &가 들어가는 명령어를 호출하는 경우 \&로 입려하거나 반드시 괄호(" /...&... ")로 호출하는 URL을 묶어야 한다.


하드웨어 정보조회
====================================

하드웨어 정보를 조회한다. :: 

   http://127.0.0.1:10040/monitoring/hwinfo
   
결과는 JSON형식으로 제공된다. ::

   {
      "version": "1.1.9",
      "method": "hwinfo",
      "status": "OK",
      "result":
      {
         "OS" : "Linux version 3.3.0 ...(생략)...", 
         "STON" : "1.1.9", 
         "CPU" : 
         { 
            "ProcCount": "4", 
            "Model": "Intel(R) Xeon(R) CPU           E5606  @ 2.13GHz", 
            "MHz": "1200.000", 
            "Cache": "8192 KB"
         }, 
         "Memory" : "8 GB", 
         "NIC" : 
         [ 
            { 
               "Dev" : "eth1", 
               "Model" : "Intel Corporation 82574L Gigabit Network Connection", 
               "IP" : "192.168.0.13", 
               "MAC" : "00:25:90:36:f4:cb" 
            } 
         ], 
         "Disk" : 
         [ 
            { 
               "Dev" : "sda", 
               "Model" : "HP DG0146FAMWL (scsi)", 
               "Total" : "238787584", 
               "Usage" : "40181760" 
            },
            {
               "Dev" : "sdb", 
               "Model" : "HITACHI HUC103014CSS600 (scsi)", 
               "Total" : "144706478080", 
               "Usage" : "2101075968" 
            },
            {
               "Dev" : "sdc", 
               "Model" : "HITACHI HUC103014CSS600 (scsi)", 
               "Total" : "144706478080", 
               "Usage" : "2012160000" 
            }
         ]
      } 
   }


재시작/종료
====================================

명령어를 통해 STON을 재시작/종료할 수 있다. 
의도하지 않은 결과를 피하기 위해 웹 페이지를 통한 확인작업이 반드시 필요하도록 개발되었다. ::

   http://127.0.0.1:10040/command/restart
   http://127.0.0.1:10040/command/restart?key=JUSTDOIT       // 즉시 실행
   http://127.0.0.1:10040/command/terminate
   http://127.0.0.1:10040/command/terminate?key=JUSTDOIT       // 즉시 실행
   

.. _getting-started-reset:

Caching 초기화
====================================

서비스를 중단하며 캐싱된 모든 컨텐츠를 삭제한다. 
설정된 모든 디스크를 포맷하며 작업이 완료되면 다시 서비스를 재개한다. ::

   http://127.0.0.1:10040/command/cacheclear
   http://127.0.0.1:10040/command/cacheclear?key=JUSTDOIT       // 즉시 실행
   
콘솔에서는 다음 명령어를 통해 전체 또는 하나의 가상호스트를 초기화한다. ::

   ./stonapi reset
   ./stonapi reset/ston.winesoft.co.kr