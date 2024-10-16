понедельник, 25 декабря 2017 г. в 16:34:13

В продолжение темы умного дома где мой [котёл умел выдавать API](https://kurapov.ee/rus/technology/gadgets/smart_home_pellet_burner_RTB30/) для мониторинга температур, захотелось мне вывести эти данные в более приятный вид. Кроме того, хотя сам котёл умеет рисовать графики, он показывал на них только температурную зону одного контура, а мне хотелось видеть два.

Поэтому я решил поизучать бесплатный dashboard и графико-генератор на основе [Grafana](http://grafana.net/). Зарегился и упёрся в то что он сам не хранит данные. Он умеет только подключаться к **внешним источникам** и делать запросы туда. 

Новый MySQL мне не хотелось поднимать, а открывать существующий тем более. Cloudwatch от Амазона - видимо полезен для мониторинга сервисов если вы активно пользуетесь этой инфраструктурой. Prometheus я могу и на работе посмотреть. Выбрал InfluxDB - специальную БД для хранения временных событий.

[Ставить Influx](https://docs.influxdata.com/influxdb/v1.4/introduction/installation/) было достаточно просто. Сложней было связать графану с influx.

Сервис гоняется на 8086 порту и Grafana хочет SSL и CORS. Поэтому пришлось делать proxy на nginx и добавлять header:  

```
location /influx/ {
    proxy_set_header HOST $host;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    add_header 'Access-Control-Allow-Credentials' 'true';

    proxy_pass http://localhost:8086/;
}
```

Теперь встаёт вопрос как данные добавить в influx. Писать будет процесс на node, поэтому я по-быстрому нашёл библиотечку и пример подключения: 

```
const Influx = require('influx');

const influx = new Influx.InfluxDB({
	host: 'localhost',
	database: 'home',
	schema: [
		{
			measurement: 'response_times',
			fields: {
				boiler_smoketemp: Influx.FieldType.FLOAT,
				boiler_output: Influx.FieldType.FLOAT,
				boiler_power: Influx.FieldType.FLOAT,
				boiler_light: Influx.FieldType.FLOAT,
				boiler_oxygen: Influx.FieldType.FLOAT,
				boiler_oxygenlow: Influx.FieldType.FLOAT,
				boiler_oxygenmid: Influx.FieldType.FLOAT,
				boiler_oxygenhigh: Influx.FieldType.FLOAT,
				boiler_connectionindex: Influx.FieldType.FLOAT,
				boiler_return: Influx.FieldType.FLOAT,
				zone_1: Influx.FieldType.FLOAT,
				zone_2: Influx.FieldType.FLOAT
			},
			tags: [
				'host'
			]
		}
	]
});

//заполняем data..

influx.writePoints([
	{
		measurement: 'heating',
		fields: data
	}
]).catch(err => {
	console.error(`Error saving data to InfluxDB! ${err.stack}`)
});
```

Теперь что-бы проверить записались ли данные, можно использовать хронограф - скачиваемый UI-сервис который в более ранних версиях был встроен в influx в виде админ-панели.

![](img/Pasted%20image%2020241016133602.png)

Окей, данные плывут, теперь осталось совсем чуть-чуть что-бы они появились и в графане где можно будет подключить и другие источники данных тоже..

![](img/Pasted%20image%2020241016133611.png)