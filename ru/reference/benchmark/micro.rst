Тест производительности микро фреймворков
=========================================

Какие тесты были выполнены?
---------------------------
Мы создали тест "Hello World", оказывающий минимальную нагрузку на каждый фреймворк. Единообразный тест выполнялся с каждым фреймворком.
Используя маршрут для HTTP метода "GET" мы передаем параметр в обработчик и возвращаем ответ "Hello $name".

Какие измерения были учтены
---------------------------
Параметры ниже были выбраны для сравнения производительности каждого фреймворка:

* Число запросов в секунду (Requests per second)
* Время на выполнение всех запросов в тесте
* Число используемых файлов на один запрос (использована функция get_included_files_).
* Использование памяти на запрос (использована функция memory_get_usage_).

Соперники
---------

* Slim_
* Silex_

Результаты
----------

Фреймворк Slim 
^^^^^^^^^^^^^^

.. code-block:: php

	# ab -n 1000 -c 5 http://localhost/bench/micro/slim/say/hello/Sonny
	This is ApacheBench, Version 2.3 <$Revision: 655654 $>
	Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
	Licensed to The Apache Software Foundation, http://www.apache.org/

	Benchmarking localhost (be patient)


	Server Software:        Apache/2.2.22
	Server Hostname:        localhost
	Server Port:            80

	Document Path:          /bench/micro/slim/say/hello/Sonny
	Document Length:        13 bytes

	Concurrency Level:      5
	Time taken for tests:   0.882 seconds
	Complete requests:      1000
	Failed requests:        0
	Write errors:           0
	Total transferred:      206000 bytes
	HTML transferred:       13000 bytes
	Requests per second:    1134.21 [#/sec] (mean)
	Time per request:       4.408 [ms] (mean)
	Time per request:       0.882 [ms] (mean, across all concurrent requests)
	Transfer rate:          228.17 [Kbytes/sec] received

	Connection Times (ms)
	              min  mean[+/-sd] median   max
	Connect:        0    4   2.4      4      33
	Processing:     0    0   0.5      0      11
	Waiting:        0    0   0.5      0      11
	Total:          2    4   2.4      4      33

	Percentage of the requests served within a certain time (ms)
	  50%      4
	  66%      4
	  75%      5
	  80%      5
	  90%      6
	  95%      8
	  98%     12
	  99%     14
	 100%     33 (longest request)

Фреймворк Silex
^^^^^^^^^^^^^^^

.. code-block:: php

	# ab -n 1000 -c 5 http://localhost/bench/micro/silex/say/hello/Sonny
	This is ApacheBench, Version 2.3 <$Revision: 655654 $>
	Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
	Licensed to The Apache Software Foundation, http://www.apache.org/

	Benchmarking localhost (be patient)


	Server Software:        Apache/2.2.22
	Server Hostname:        localhost
	Server Port:            80

	Document Path:          /bench/micro/silex/say/hello/Sonny
	Document Length:        12 bytes

	Concurrency Level:      5
	Time taken for tests:   2.228 seconds
	Complete requests:      1000
	Failed requests:        0
	Write errors:           0
	Total transferred:      225000 bytes
	HTML transferred:       12000 bytes
	Requests per second:    448.75 [#/sec] (mean)
	Time per request:       11.142 [ms] (mean)
	Time per request:       2.228 [ms] (mean, across all concurrent requests)
	Transfer rate:          98.60 [Kbytes/sec] received

	Connection Times (ms)
	              min  mean[+/-sd] median   max
	Connect:        0   11   5.1     10      44
	Processing:     0    0   1.1      0      26
	Waiting:        0    0   1.1      0      26
	Total:          5   11   5.1     10      45

	Percentage of the requests served within a certain time (ms)
	  50%     10
	  66%     12
	  75%     13
	  80%     14
	  90%     17
	  95%     20
	  98%     25
	  99%     29
	 100%     45 (longest request)

Phalcon 0.5.0
^^^^^^^^^^^^^

.. code-block:: php

	# ab -n 1000 -c 5 http://localhost/bench/micro/phalcon/say/hello/Sonny
	This is ApacheBench, Version 2.3 <$Revision: 655654 $>
	Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
	Licensed to The Apache Software Foundation, http://www.apache.org/

	Benchmarking localhost (be patient)


	Server Software:        Apache/2.2.22
	Server Hostname:        localhost
	Server Port:            80

	Document Path:          /bench/micro/phalcon/say/hello/Sonny
	Document Length:        12 bytes

	Concurrency Level:      5
	Time taken for tests:   0.397 seconds
	Complete requests:      1000
	Failed requests:        0
	Write errors:           0
	Total transferred:      205000 bytes
	HTML transferred:       12000 bytes
	Requests per second:    2516.74 [#/sec] (mean)
	Time per request:       1.987 [ms] (mean)
	Time per request:       0.397 [ms] (mean, across all concurrent requests)
	Transfer rate:          503.84 [Kbytes/sec] received

	Connection Times (ms)
	              min  mean[+/-sd] median   max
	Connect:        0    2   0.9      2      11
	Processing:     0    0   0.2      0       5
	Waiting:        0    0   0.2      0       4
	Total:          1    2   0.9      2      11

	Percentage of the requests served within a certain time (ms)
	  50%      2
	  66%      2
	  75%      2
	  80%      2
	  90%      3
	  95%      4
	  98%      5
	  99%      5
	 100%     11 (longest request)



Графики
^^^^^^^
Первый график показывает, сколько запросов в секунду смог принять каждый фреймворк. Второй график показывает среднее время выполнения всех запросов.


.. raw:: html

	<script type="text/javascript" src="https://www.google.com/jsapi"></script>
	<script type="text/javascript">
		google.load("visualization", "1", {packages:["corechart"]});
		google.setOnLoadCallback(drawChart);

		function drawChart() {

			var data = new google.visualization.DataTable();
			data.addColumn('string', 'Framework');
			data.addColumn('number', 'Requests per second');
			data.addRows([
				['Silex',    448.75],
				['Slim',    1134.21],
				['Phalcon', 2516.74]
			]);

			var options = {
				title: 'Framework / Requests per second (#/sec) [more is better]',
				colors: ['#3366CC'],
				animation: {
					duration: 0.5
				},
				fontSize: 12,
				chartArea: {
					width: '600px'
				}
			};

			var chart = new google.visualization.ColumnChart(document.getElementById('rps_div'));
			chart.draw(data, options);

			var data = new google.visualization.DataTable();
			data.addColumn('string', 'Framework');
			data.addColumn('number', 'Time per Request');
			data.addRows([
				['Silex',   2.228],
				['Slim',    0.882],
				['Phalcon', 0.397]
			]);

			var options = {
				title: 'Framework / Time per Request (mean, across all concurrent requests) [less is better]',
				colors: ['#3366CC'],
				fontSize: 11
			};

			var chart = new google.visualization.ColumnChart(document.getElementById('tpr_div'));
			chart.draw(data, options);

			var data = new google.visualization.DataTable();
			data.addColumn('string', 'Framework');
			data.addColumn('number', 'Memory Usage (MB)');
			data.addRows([
				['Silex',   1.25],
				['Slim',    1.25],
				['Phalcon', 0.75]
			]);

			var options = {
				title: 'Framework / Memory Usage (mean, megabytes per request) [less is better]',
				colors: ['#3366CC'],
				fontSize: 11
			};

			var chart = new google.visualization.ColumnChart(document.getElementById('mpr_div'));
			chart.draw(data, options);

			var data = new google.visualization.DataTable();
			data.addColumn('string', 'Framework');
			data.addColumn('number', 'Number of included PHP files');
			data.addRows([
                ['Silex',    54],
				['Slim',     17],
				['Phalcon',   2]
			]);

			var options = {
				title: 'Framework / Number of included PHP files (mean, number on a single request) [less is better]',
				colors: ['#3366CC'],
				fontSize: 11
			};

			var chart = new google.visualization.ColumnChart(document.getElementById('nfi_div'));
			chart.draw(data, options);

		}
	</script>
	<div align="center">
		<div id="rps_div" style="width: 600px; height: 400px; position: relative; "><iframe name="Drawing_Frame_31166" id="Drawing_Frame_31166" width="600" height="400" frameborder="0" scrolling="no" marginheight="0" marginwidth="0"></iframe><div></div></div>
		<div id="tpr_div" style="width: 600px; height: 400px; position: relative; "><iframe name="Drawing_Frame_89467" id="Drawing_Frame_89467" width="600" height="400" frameborder="0" scrolling="no" marginheight="0" marginwidth="0"></iframe><div></div></div>
		<div id="nfi_div" style="width: 600px; height: 400px; position: relative; "><iframe name="Drawing_Frame_49746" id="Drawing_Frame_49746" width="600" height="400" frameborder="0" scrolling="no" marginheight="0" marginwidth="0"></iframe><div></div></div>
		<div id="mpr_div" style="width: 600px; height: 400px; position: relative; "><iframe name="Drawing_Frame_77939" id="Drawing_Frame_77939" width="600" height="400" frameborder="0" scrolling="no" marginheight="0" marginwidth="0"></iframe><div></div></div>
	</div>

Заключение
----------
Уникальная структура Phalcon предоставляет исключительную производительность и превосходит все используемые в этом тесте фреймворки.

.. _get_included_files: http://www.php.net/manual/en/function.get-included-files.php
.. _memory_get_usage: http://php.net/manual/en/function.memory-get-usage.php
.. _Slim: http://slimframework.com/
.. _Silex: http://silex.sensiolabs.org/
