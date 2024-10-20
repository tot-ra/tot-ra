вторник, 9 февраля 2016 г. в 11:25:15

Я не так давно писал о [protractor'е](http://kurapov.ee/rus/lab/quality_control/frontend/protractor/) — он облегчает работу с selenium, хоть и завязан с jasmine. Недавно вышла 3я версия и я, заодно обновив jasmine до 2.4.1, решил двинуться дальше со сценарными тестами, а именно - лишиться их, разбив на изолированные UI-тесты. Зачем?

Когда вы пишете сценарные тесты, то довольно скоро сталкиваетесь с проблемой заготовки данных. В интеграционных тестах я уже решал эту проблему, используя [отдельную базу данных на каждый тест](http://kurapov.ee/rus/lab/quality_control/phpunit/isolated_db_for_test/). Это довольно тяжело и энергоёмко. Когда дело касается UI-тестов, это ещё сложней.   

### Сервис и QA API

Я написал специальный сетевой API, который заготавливает данные на backend и специальный объект на frontend, который помогает заготавливать данные. Под каждый тип данных у меня есть метод

```
process.env.NODE_TLS_REJECT_UNAUTHORIZED = "0";
var request                              = require('request');
exports.testData = {

	newArticle: function (callback) {
		return request.get('https://kurapov.ee/dataForUITest/newArticle',function (error, response, body) {
			callback(JSON.parse(body));
		});
	},
	
	//more actions go here
};
```

На backend используются существующие модели и заготовленные тексты, что-бы избежать постоянно меняющихся SQL. Получается API похожий на публичный, только методы не удаляют ничего, а занимаются только заготовкой  

### Последствия на frontend

Из плюсов

- тесты стали стабильней, можно перезапускать индивидуальные тесты без всех зависимостей
- тесты можно распараллеливать, если они не сильно привязаны к навигации по меняющимся в это же время UI-элементам (меню с сортировкой по времени изменения)
- тесты проще, т.к. тестируют конкретный модуль/сущность которая так же создаётся на backend, вместо сценарного тестирования работы «всего»

Из недостатков

- Асинхронные тесты. Так как jasmine не знает что делается сетевой запрос, то в тест приходится вносить **done** функцию и вызывать её когда тест закоченен. С этим накладываются другие проблемы, как то проверка консоли на ошибки
- Тесты более изолированные и не тестируют навигацию по UI-элементам, а больше акцентируются на URL'ы. Поэтому если какие-то виды запрятаны глубоко с одним и тем же URL'ом, то последовательность операций не радует

```
it('File editing', function (done) {
	dataForTesting.getFile(function (exFile) {
		myPageWrapper.loginAs('artjom@kurapov.ee', function () {
			page.saveButtonShouldBeDisabled();

			page.file.editTitle().then(function () {
				page.saveForm();
				browser.driver.sleep(200).then(done);
			});
		});
	});
});
```


<iframe width="934" height="350" src="https://www.youtube.com/embed/n3APDzvPF0k" title="2015.12.29 Никита Макаров - Микросервисы для автоматизации тестирования («Одноклассники»)" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
