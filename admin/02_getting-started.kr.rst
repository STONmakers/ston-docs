.. _getting-started:

시작하기
******************

모든 것이 간단합니다. 빠르게 설치해서 바로 서비스에 투입할 수 있습니다. 이 장에서는 STON을 설치하고 간단한 가상호스트를 구성하여 실행하는 것까지의 내용을 다룹니다.

.. toctree::
   :maxdepth: 2


설치하기
====================================
STON은 표준 Linux 서버에서 무리없이 동작하도록 개발되었습니다. 특정 하드웨어에 
종속적인 기능은 최대한 배제하였습니다. 이는 고객이 합리적인 장비 선택을 할 수 있도록 
돕는 것 또한 STON의 목표이기 때문입니다. 서비스의 특성과 규모에 따라 적절한 서버를 
구성하는 것이 서비스의 시작입니다.


서버 구성
-----------------------------------------------

일반적인 서버 선택의 기준은 CPU, 메모리, 디스크입니다. 10Gbps급의 높은 퍼포먼스를
요구하는 서비스의 경우 각 구성요소가 서비스 특성을 만족해야 합니다.

-  **CPU**

   STON은 Many-Core에 대해 확장성(Scalability)를 가집니다. 코어가 많으면 많을수록 
   초당 처리량이 증가합니다. 높은 처리량이 반드시 높은 트래픽을 의미하는 것은 아닙니다.

   .. figure:: img/10g_cpu.jpg
      :align: center

      클라이언트가 많을수록 많은 CPU는 힘이 됩니다.
    
   위 그림처럼 평균파일 크기가 4KB인 서비스는 약 26만번을 전송해야 1GB파일을 
   한번 전송한 것과 같은 대역폭을 사용합니다. CPU선택의 가장 큰 기준은 얼마나 
   많은 동시접속을 처리하는가에 있습니다. 일반적으로 Quad 코어 이상이 적합합니다. 
   

-  **메모리**

   최소 4GB이상을 권장합니다. 자주 접근되는 콘텐츠를 메모리에 상주시켜야 
   성능이 향상되는데, 물리적인 메모리가 작다면 디스크 부하가 가중됩니다.
   파일이 많지 않더라도 콘텐츠 크기가 커서 디스크 I/O가 높다면 메모리를 
   증설하는 것이 좋은 해결책입니다.


-  **디스크**

   OS를 포함하여 최소 3개 이상을 권장합니다. 디스크 역시 많으면 많을수록 많은 
   콘텐츠를 캐싱할 수 있을 뿐만 아니라 부하를  분산할 수 있으므로 좋습니다.
   
   .. figure:: img/02_disk.png
      :align: center
      
   OS와 STON은 별도의 디스크로 구성하시기 바랍니다.
   
   일반적으로 OS가 설치된 디스크에 STON을 설치합니다. 로그 역시 같은 디스크에
   구성하는 것이 일반적입니다. (로그의 경우 모든 서비스 상황을 실시간으로 기록하기 
   때문에 항상 Write작업이 있습니다.) STON은 디스크를 RAID 0처럼 사용합니다. 
   퍼포먼스와 RAID의 상관여부는 고객 서비스 특성에 따라 달라지므로 단언할 수 없습니다. 
   파일 변경이 빈번하지 않고 컨텐츠의 크기가 메모리 캐싱크기보다 훨씬 큰 경우 
   RAID를 통한 Read속도 향상이 도움이 될 수 있습니다.



OS 구성
-----------------------------------------------

표준 64bit Linux 배포판(Cent 6.2이상, Ubuntu 10.04이상)에서 동작합니다.
가장 기본적인 형태로 설치하시면 됩니다. STON은 다른 패키지에 종속성을 
가지지 않으므로 추가로 설치하실 것은 없습니다.



설치
-----------------------------------------------

1. 최신버전의 STON을 다운로드 받는다. ::

      [root@localhost ~]# wget  http://webhard.winesoft.co.kr/ston/ston.2.0.0.rhel.2.6.32.x64.tar.gz
      --2014-06-17 13:29:14--  http://webhard.winesoft.co.kr/ston/ston.2.0.0.rhel.2.6.32.x64.tar.gz
      Resolving webhard.winesoft.co.kr... 192.168.0.14
      Connecting to webhard.winesoft.co.kr|192.168.0.14|:80... connected.
      HTTP request sent, awaiting response... 200 OK
      Length: 71340645 (68M) [application/x-gzip]
      Saving to: “ston.2.0.0.rhel.2.6.32.x64.tar.gz”
      
      100%[===============================================>] 71,340,645  42.9M/s   in 1.6s
      
      2014-06-17 13:29:15 (42.9 MB/s) - “ston.2.0.0.rhel.2.6.32.x64.tar.gz” saved [71340645/71340645]


2. 압축을 해지합니다. ::

		[root@localhost ~]# tar -zxf ston.2.0.0.rhel.2.6.32.x64.tar.gz

3. 설치 스크립트를 실행합니다. ::

		[root@localhost ~]# ./ston.2.0.0.rhel.2.6.32.x64.sh

4. 설치과정은 install.log에 기록됩니다. 로그를 통해 설치 중 발생하는 문제를 찾아낼 수 있습니다. ::

      #DownloadURL: http://webhard.winesoft.co.kr/ston/ston.2.0.0.rhel.2.6.32.x64.tar.gz
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
최신버전이 배포되면 stonu명령어로 업데이트할 수 있습니다. ::

	./stonu 2.0.1
	
또는 WM(Web Management)을 통해 업데이트를 진행할 수 있습니다.

   .. figure:: img/conf_update1.png
      :align: center


실행하기
====================================

실행
-----------------------------------------------
기본 설치경로는 /usr/local/ston/ 입니다. 최초 설치시 필수 설정파일인 server.xml과 
vshost.xml이 존재하지 않습니다. 설치 경로의 server.xml.default와 vhosts.xml.default를 
복사 또는 수정하시어 설정하시기 바랍니다. *.default파일은 항상 최신패키지와 함께 배포됩니다.

1. 발급받은 license.xml을 설치 경로에 복사합니다.

2. server.xml을 열어 <Storage>를 구성합니다. ::

      <Server>
          <Cache>
              <Storage>
                  <Disk>/cache1/</Disk>
                  <Disk>/cache2/</Disk>
              </Storage>
          </Cache>
      </Server>
      
.. note::

   STON은 기본적으로 디스크를 저장공간으로 사용하기 때문에 디스크가 설정되어 있지 않으면
   구동되지 않습니다. 자세한 내용은 다음 장에서 설명합니다. ::

3. STON을 실행합니다.  ::

		[root@localhost ~]# service ston start

4. STON을 중지합니다.  ::

		[root@localhost ~]# service ston stop




      
