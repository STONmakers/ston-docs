.. _api-graph:

/Graph
******************

그래프 관련 API목록.

.. toctree::
   :maxdepth: 2
   

모든 그래프는 PNG포맷으로 제공된다. 
호출 규칙은 자원 뒤에 단위가 붙는 형식이다. ::

    # CPU의 경우 다음과 같이 5가지의 API가 제공됩니다.
    http://127.0.0.1:10040/graph/cpu_dash.png
    http://127.0.0.1:10040/graph/cpu_day.png
    http://127.0.0.1:10040/graph/cpu_week.png
    http://127.0.0.1:10040/graph/cpu_month.png
    http://127.0.0.1:10040/graph/cpu_year.png
    
모든 그래프는 5가지 타입으로 제공된다.

======= =========== =========== =============
타입	크기	    평균단위	기간
======= =========== =========== =============
dash	205 X 175	5분	        12시간
day	    580 X 203	5분         2일 (48시간)
week	580 X 203	30분	    2주 (14일)
month	580 X 203	2시간	    7주
year	580 X 203	1일	        18개월
======= =========== =========== =============

한 그래프에는 최소 1개에서 최대 3개의 선이 그려진다. 
Main Line은 녹색, Sub Line은 파란색으로 그려진다. 
또한 "Week" 그래프 이상부터는 Peak Line이 제공된다. 
Peak Line은 이전 단위에서 가장 큰 수치를 핑크색으로 그린다.

.. note:
   
   너무 많은 그래프를 동시에 그릴 경우 CPU사용량이 과도하게 높아져 서비스 품질저하가 발생할 수 있다. 
   이를 방지하기 위해 항상 한번에 하나의 그래프만 그리도록 관리한다.



.. _api-graph-global:

전역자원 그래프
====================================

전역자원 그래프는 시스템 상태 또는 STON과 관련된 자원들에 대해 서비스한다. 
아래 표에서 *는 타입(dash, day, week, month, year) 중 한 가지를 의미한다.

   .. figure:: img/graph_global.png
      :align: center
      
      
      
.. _api-graph-vhost:

가상호스트 그래프
====================================

가상호스트 그래프는 전체 또는 개별 가상호스트의 상태에 대해 서비스한다. 
vhost파라미터를 이용하여 특정 가상호스트를 지정할 수 있으며, 
생략된 경우 전체 가상호스트의 합을 제공한다. ::

    http://127.0.0.1:10040/graph/vhost/mem_day.png?vhost=example.com
    
아래 표에서 *는 타입(dash, day, week, month, year) 중 한 가지를 의미한다.

   .. figure:: img/graph_vhost.png
      :align: center