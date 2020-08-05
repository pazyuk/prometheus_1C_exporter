# prometheus_1C_exporter

Приложение выполняет роль explorer'а для prometheus. На текущий момент приложение собирает метрики:
* Используемые клиентские лицензии
* Доступную производительность серверов приложений
* Количество соединений
* Количество сеансов
* Текущая память процесса (получается из ОС, пока поддерживается только linux)
* Проверка галки "блокировка регламентных заданий"
* Память всего
* Память текущая
* Чтение/Запись текущее
* Время вызова текущее
* Время вызова СУБД
* Процессорное время текущее

сборка показателей осуществляется через утилиту rac.
Каждую из метрик можно ставить на паузу, например такое может потребоваться в процессе обновления ИБ т.к. соединения RAC могут мешать этому процессу. Что бы поставить на паузу нужно отправить GET запрос

http://host:9091/Pause?metricNames=ProcData,SessionsMemory&offsetMin=1

где **metricNames** это метрики через запятую, **offsetMin** это пауза в минутах после которой автоматически включается сбор показателей. offsetMin - необязательный, если его не указывать сбор будет приостановлен будет пока явно его не запустить, запуск производится так:

http://host:9091/Continue?metricNames=ProcData,SessionsMemory

Имена метрик можно посмотреть в конфиге **settings.yaml**




**Запуск** 

./1C_exporter -port=9095 --settings=/usr/local/bin/settings.yaml

Если порт не указать по дефолту будет порт 9091




в конфииге прометеуса (prometheus.yml) нужно указать хосты на которых запущен explorer
```yaml
  - job_name: '1С_Metrics'
    metrics_path: '/1С_Metrics' 
    static_configs:
    - targets: ['host1:9091', 'host2:9091', 'host3:9091', 'host4:9091']
```
```golang
end:
```
Все, настраиваем дажборды, умиляемся. 

------------



Если захотите развить explorer, что бы собирались другие метрики, нужно:
Создать файл [name metrics].go в котором будет описан класс метрики, класс должен имплементировать интерфейс Iexplorer, после чего добавляем экземпляр класса к метрикам:
```golang
metric := new(Metrics)
	metric.append(new(ExplorerClientLic).Construct(time.Second * 10))
	// metric.append(новый explorer объект) 
	
	for _, ex := range metric.explorers {
		go ex.StartExplore()
}
```
```golang
goto end
```

**Примеры дажбордов**

![](doc/img/browser_d8CBonI15Y.png "")

![](doc/img/browser_FCaSoFVBDe.png "Доступная производительность")

![](doc/img/browser_jtYHlI4MPZ.png "Память по сеансам")

![](doc/img/browser_LnTYeIKxgG.png "Клиентские лицензии")
