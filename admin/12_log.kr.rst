.. admin-log:

로그
******************

로그는 전역과 가상호스트로 구분된다. 
모든 로그는 기록여부를 설정할 수 있으며, 공통된 속성을 가지고 있다. ::

    <XXX Type="time" Unit="1440" Retention="10">ON</XXX>

-  ``Type (기본: time)`` , ``Unit (기본: 1440분)`` 로그 롤링조건을 설정한다.

   - ``time`` 설정된 ``unit`` 시간(단위: 분)마다 로그 파일을 롤링한다.
   - ``size`` 설정된 ``unit`` 크기(단위: MB)마다 로그 파일을 롤링한다.
   - ``both`` 콤마(,)로 구분하여 시간과 크기를 동시에 설정한다.
     예를 들어 Unit="1440, 100"인 경우 시간이 24시간(1440분) 또는 100MB 인 경우 로그 파일을 롤링한다.
     
-  ``Retention (기본: 10개)`` 단위 로그파일을 최대 n개 유지한다.

``Type`` 이 "time" , ``Unit`` 이 10이면 로그는 매 10분에 롤링된다.
예를 들어 서비스를 2:18분에 시작해도 로그는 매 10분인 2:20, 2:30, 2:40에 롤링된다. 
마찬가지로 하루에 한번 매일 0시 0분에 롤링하려면 1440(60분 X 24시)으로 ``Unit`` 값으로 설정한다. 
``time`` 설정에서 로그는 하루에 한번 무조건 롤링되므로 ``Unit`` 의 최대값은 1440을 넘을 수 없다.

.. figure:: img/log_rolling1.jpg
   :align: center
   
최대 값인 24시간(Unit=1440)시간마다 로그가 롤링되도록 설정했다면 다음 그림과 같이 로그가 기록된다.

.. figure:: img/log_rolling2.jpg
   :align: center


.. toctree::
   :maxdepth: 2



.. admin-log-install:

설치로그
====================================

STON의 설치/업그레이드 시 모든 내용이 install.log에 기록된다. 
이 로그는 별도의 설정이 없다. ::

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


.. admin-log-global:


전역로그
====================================

전역로그는 전역설정(server.xml)에 설정한다. ::

    <Cache>
        <InfoLog Type="size" Unit="1" Retention="5">ON</InfoLog>
        <DenyLog Type="size" Unit="1" Retention="5">ON</DenyLog>
        <OriginErrorLog Type="size" Unit="5" Retention="5" Warning="OFF">ON</OriginErrorLog>      
    </Cache>

-  ``<InfoLog> (기본: ON, Type: size, Unit: 1)``
   
   STON의 동작과 설정변경에 대해 기록한다.

-  ``<DenyLog> (기본: ON, Type: size, Unit: 1)``

   :ref:`access-control-serviceaccess` 에 의해 접근차단된 IP를 기록한다. ::
   
      #Fields: date time c-ip deny
      2012.11.15 07:06:10 1.1.1.1 AP
      2012.11.15 07:06:26 2.2.2.2 GIN
      2012.11.15 07:06:30 3.3.3.3 3.3.3.1-255
      
   모든 필드는 공백으로 구분되며 각 필드의 의미는 다음과 같다.
   
   - ``date`` 날짜
   - ``time`` 시간
   - ``c-ip`` 클라이언트 IP
   - ``deny`` 차단조건
   

-  ``<OriginErrorLog> (기본: OFF, Type: size, Unit: 5, Warning: OFF)``

   모든 가상호스트의 원본서버에서 발생한 장애만을 기록한다. 
   장애는 접속장애와 전송장애를 의미하며 원본서버 배제/복구 결과가 기록된다. ::
   
      #Fields: date time vhostname level s-domain s-ip cs-method cs-uri time-taken sc-error sc-resinfo
      2012.11.15 07:06:10 [example.com] [ERROR] 192.168.0.13 192.168.0.13 GET /Upload/ProductImage/stock/1716439_SM.jpg 20110 Connect-Timeout -
      2012.11.15 07:06:26 [example.com] [ERROR] 192.168.0.13 192.168.0.13 GET /Upload/ProductImage/stock/1716439_SM.jpg 20110 Connect-Timeout -
      2012.11.15 07:06:30 [example.com] [ERROR] 192.168.0.13 192.168.0.13 GET /Upload/ProductImage/stock/1716439_SM.jpg 20110 Connect-Timeout -
      #2012.11.15 07:06:30 [example.com] 192.168.0.13 excluded from service
      #2012.11.15 07:06:31 [example.com] Origin server list: 192.168.0.14
      #2012.11.15 07:11:11 [example.com] 192.168.0.13 recovered back in service
      #2012.11.15 07:11:12 [example.com] Origin server list: 192.168.0.13
   
   모든 필드는 공백으로 구분되며 각 필드의 의미는 다음과 같다.
   
   - ``date`` 장애발생 날짜
   - ``time`` 장애발생 시간
   - ``vhostname`` [가상호스트]
   - ``level`` [장애레벨(Error 또는 Warning)]
   - ``s-domain`` 원본서버 도메인
   - ``s-ip`` 원본서버 IP
   - ``cs-method`` STON이 원본서버에게 보낸 HTTP Method
   - ``cs-uri`` STON이 원본서버에게 보낸 URI
   - ``time-taken`` 장애가 발생 할때 까지 소요된 시간
   - ``sc-error`` 장애의 종류
   - ``sc-resinfo`` 장애발생시 서버 응답 정보(","문자로 구분)
   
   ``Warning`` 속성이 ``ON`` 이라면 다음과 잘못된 HTTP통신이 발생한 경우에 기록한다. ::
   
      2012.11.15 07:09:03 [example.com] [WARNING] 10.10.10.10 121.189.63.219 GET /716439_SM.jpg 20110 PartialResponseOnNormalRequest Res=206,Len=2635
      2012.11.15 07:09:03 [example.com] [WARNING] 10.10.10.10 121.189.63.219 GET /716439_SM.jpg 20110 ClosedWithoutResponse -
      
   잘못된 HTTP통신의 경우는 다음과 같다.
   
   - ``ClosedWithoutResponse`` 원본서버에 의한 연결종료. HTTP 응답을 받지 못했다.
   - ``ClosedWhenDownloading`` 원본서버에 의한 연결종료. Content-Length 만큼 다운로드하지 못했다.
   - ``NotPartialResponseOnRangeRequest`` Range요청을 했으나 응답코드가 206이 아니다.
   - ``DifferentContentLengthOnRangeRequest`` 요청한 Range와 Content-Length가 다르다.
   - ``PartialResponseOnNormalRequest`` Range요청이 아닌데 응답코드가 206이다.



.. admin-log-syslog:

SysLog 전송
---------------------

`syslog <http://en.wikipedia.org/wiki/Syslog>`_ 프로토콜을 사용하여 로그를 UDP로 실시간 포워딩한다. 
모든 로그에 대하여 syslog로 전송되도록 설정할 수 있다. ::

    <Cache>
        <InfoLog SysLog="OFF">ON</InfoLog>
        <DenyLog SysLog="OFF">ON</DenyLog>
        <OriginErrorLog SysLog="OFF">ON</OriginErrorLog>
    </Cache>
    
-  ``SysLog``

   - ``OFF (기본)`` syslog를 사용하지 않는다.
   
   - ``ON`` 이 태그 하위에 설정된 ``<SysLog>`` 로 로그를 전송한다.
   
다음은 ``<OriginErrorLog>`` 가 기록될 때 syslog를 설정하는 예제이다. ::

    <OriginErrorLog SysLog="ON">
        <SysLog Priority="local3.info" Dest="192.168.0.1:514" />
        <SysLog Priority="user.alert" Dest="192.168.0.2" />
        <SysLog Priority="mail.debug" Dest="log.example.com" />
    </OriginErrorLog>
    
1. ``<OriginErrorLog>`` 의 ``SysLog`` 속성을 ``ON`` 으로 설정한다.
#. ``<OriginErrorLog>`` 의 하위에 ``<SysLog>`` 태그를 생성한다. n대의 서버로 동시에 전송가능하다.
#. ``<SysLog>`` 의 ``Priority`` 속성을 설정한다. 
   이 표현은 syslog의 `Facility Levels <http://en.wikipedia.org/wiki/Syslog#Facility_levels>`_ 과 
   `Severity levels <http://en.wikipedia.org/wiki/Syslog#Severity_levels>`_ 의 조합으로 구성한다.
#. ``<SysLog>`` 의 ``Dest`` 속성을 설정한다. syslog수신서버를 의미하며 수신포트가 514인 경우 생략가능하다.

위 설정으로 기록된 sys로그 예제는 다음과 같다. 
syslog의 tag는 STON/{로그명}으로 기록된다. ::

    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: 2013-03-12 14:09:20 [ERROR] [example.com] - 192.168.0.14 GET /1.gifd 1996 Connect-Timeout -
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: 2013-03-12 14:09:22 [ERROR] [example.com] - 192.168.0.14 GET /favicon.ico 1995 Connect-Timeout -
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: 2013-03-12 14:09:24 [ERROR] [example.com] - 192.168.0.14 GET /1.gifd22 2020 Connect-Timeout -
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: #2013 .03.12 14:09:24 [example.com] 192.168.0.14:102 excluded from service
    Mar 12 11:24:24 192.168.0.1 STON/ORIGINERROR: #2013 .03.12 14:09:24 [example.com] Origin server list:
    
