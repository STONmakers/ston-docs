.. _api-reference:

API Reference
**********************

간단한 HTTP통신을 통해 API를 호출할 수 있다.
브라우저에 주소를 입력하는 것만으로도 모든 기능을 사용할 수 있도록 개발되었다.
ACL(Access Control List)에 의해 허가받지 않은 요청이 발생하면 해당 세션은 곧바로 
종료된다.

모든 API는 다음과 같이 콘솔에서 커맨드로 수행이 가능하다. ::

    ./stonapi help
    ./stonapi conf/reload
    ./stonapi command/expireafter?sec=86400\&url=winesoft.co.kr
    
    
HTTP API는 &를 QueryString의 구분자로 인식하지만 Linux 콘솔에서는 다른 의미를 가진다. 
그러므로 &가 들어가는 명령어를 호출할 때는 \&로 입려하시거나 반드시 괄호(" /...&... ")로 
QQ :_api_conf:_api_conf_show:`TEST TEST` QQQQQ
호출 URL을 묶어야 한다.

Contents:

.. toctree::
   :maxdepth: 2

   conf.kr
   command.kr
   monitoring.kr   
   graph.kr
   etc.kr
   
  

