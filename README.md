<h1>Welcome to my portfolio!</h1>

## 📚 Содержание

- [Сокращенная часть документа Стандарт API - техпис](#Standart_API)
- [Описание логики разработки](#Use_case_description)
- [Описание функционала системы](#Functional_requirements)
- [Моделирование бизнес-процессов (BPMN)](#BPMN)
- [Диаграмма деятельности](#Activity_diagram)
- [Диаграмма последовательности](#Sequence_diagram)
- [Диаграмма Use case](#Use_case)
- [Интеграции ИС](#API)
- [Прототипирование интерфейсов](#Prototypes)
- [Дипломный проект](#Diploma)
- [Data Warehouse](#DWH)
- [SQL](#SQL)
- [BI (Tableau)](#BI)


## Standart_API ТЕХПИС ⬇
<details><summary>Сокращенная часть документа Стандарт API</summary>
	
## API design mandate

1. Оперируйте сущностями, а не имплементациями
1. Используйте операции для отображения статуса асинхронных действий
1. Думайте о безопасности своих интерфейсов так, как будто они доступны из интернета
1. Не используйте недокументированные интерфейсы (API) других команд
1. Не используйте API для которых не предоставляются гарантии доступности и совместимости
1. Не проксируйте в свой интерфейс сущности своих зависимостей
1. Не предоставляйте методы API меняющие состояние без авторизации
1. Не используйте RPC семантику в URL, оперируйте только REST сущностями или операциями
1. Используйте паттерн -- при выполнении сложных многосоставных операций
1. Используйте возврат объектов отслеживания асинхронных операций 
1. Все длительные асинхронные операции следует делать прерываемыми 
1. Не возвращайте в API ошибки, если пользовательский сценарий не окончен и их невозможно обработать

## Введение

**API** – это спецификация, описывающая как клиент должен формировать запросы на получение и изменение ресурсов, а сервер отвечать на эти запросы.

## Соглашения

В документах описывающих стандарты, используются модальные глаголы для обозначения уровня требований. Такие слова выделяются заглавными буквами. В этом документе определяется толкование этих глаголов и производных от них слов. 

- *НЕОБХОДИМО*

Или ТРЕБУЕТСЯ, НУЖНО и ДОЛЖЕН – для требований, которые являются абсолютно необходимыми в данном соглашении.

- *НЕДОПУСТИМО*

Или НЕ ПОЗВОЛЯЕТСЯ – абсолютный запрет в рамках соглашения.

- *СЛЕДУЕТ*

Или РЕКОМЕНДУЕТСЯ – для обозначения требований, от выполнения которых можно отказаться при наличии причин. Однако при этом, следует помнить о возможных проблемах и принимать взвешенное решение.

- *НЕ СЛЕДУЕТ*

Или НЕ РЕКОМЕНДУЕТСЯ – применительно к особенностям или функциям, которые допустимы и могут быть полезными, но могут вызывать проблемы. При реализации таких опций следует принимать взвешенное решение.

- *ВОЗМОЖНО*

Или НЕОБЯЗАТЕЛЬНЫЙ – обозначают элементы, реализация которых является необязательной. 

## Структура документа

В этом разделе описывается структура json-документа.

Если не указано иное, определенные здесь объекты, НЕ ДОЛЖНЫ содержать дополнительных параметров. Реализации клиента и сервера ДОЛЖНЫ игнорировать параметры, отличные от указанных.

### Типы параметров

|Тип |Описание|
|---|---|
|строка|строковое значение произвольной длины|
|число|целое число или число с плавающей точкой|
|Array&lt;type&gt;|Массив объектов типа type|
|null|Неопределенное (пустое) значение|
|boolean|Булево значение (true/false)|

### Общая структура документа

Документ должен представлять собой корректный JSON-документ описанный в спецификации --.

На верхнем уровне – JSON может являться представлением одного из допустимых встроенных типов объекта. Также, может являться определяемым данной спецификацией, или одним из типов ресурса определенных сервером.

#### Resource object

Resource object – базовый блок всех объектов описанных в спецификации и любых объектов определенных сервером. Все объекты унаследованные от resource object определяют два типа атрибутов:

- *Служебные* – определяют данные необходимые для получения и изменения объекта. Содержат мета-информацию об объекте, либо выполняют какую-либо функцию. Служебные атрибуты, не являются частью состояния ресурсов на сервере. Служебные атрибуты выносятся на верхний уровень в объекте.

- *Сущностные* – описывают свойства доменных сущностей сервера и не могут нести в себе служебную информацию. Описываются внутри **attributes** свойства находящегося на верхнем уровне в ресурсе. Если сущностных атрибутов нет, то и атрибута **attributes** также не будет.

Любой из встроенных или определенных сервером объектов наследуется от Resource object и ДОЛЖЕН определять параметры:

|Параметр|Тип| Описание|
|---|---|---|
|kind|строка|Тип объекта, описанный латиницей в CapitalCase, например &#34;Page&#34;, &#34;Collection&#34;, &#34;Error&#34;, &#34;Document&#34;|

МОГУТ быть определены параметры:

|Параметр|Тип|Описание|
|---|---|---|
|self|строка (URL)|Абсолютная ссылка на ресурс. Запрос сделанный по данной ссылке возвращает представление данного ресурса|
|metadata|JSON|Дополнительная служебная информация. Произвольный JSON. В metadata вносят служебные данные, которые могут быть полезны на клиенте в контексте выполняемого запроса, но: <br /> 1) определяются исключительно на сервере, и не могут быть изменены запросом с клиента<br /> 2) не являются непосредственными атрибутами ресурса<br /> Например, дата/время создания или изменения ресурса, юзер создавший ресурс, и т.п.|
|schema|строка URL [JSON schema]|Ссылка на схему описывающую ресурс в формате JSON schema|
|attributes|JSON|Описание сущностных атрибутов ресурса|
|dependencies|JSON|Связи объекта вида providedBy, usedBy|

#### Встроенные типы объектов

<details>
<summary>Ссылка (kind=Reference)</summary>

В случаях, когда необходимо возвращать лишь уникальный идентификатор описывающий объект, используется resource reference object. Каждый объект Reference ДОЛЖЕН определять параметры:

|Параметр|Обязательный|Описание|
|--|--|--|
|kind|строка = Reference|да|Тип ресурса. В этом случае это тип Reference|
|resourceKind|строка|да|Тип ресурса на который ведет ссылка|
|identifier|AttributeIdentifier|да|Идентификатор ресурса (см ниже)|

AttributeIdentifier

|Параметр|Тип|Обязательный|Описание|
|--|--|--|--|
|name|строка|да|Имя уникального атрибута в ресурсе|
|value|строка/число|да|Значение идентификатора|

</details>
<details>
<summary>Ошибка(kind=Error)</summary> 

Все ошибки сервера, ошибки бизнес-логики или инфраструктуры, ДОЛЖНЫ возвращаться в ресурсе с типом kind=Error. НЕ ДОПУСКАЕТСЯ возвращать данный тип ресурса вместе с кодом ответа HTTP &lt; 400

|Параметр|Тип|Обязательный|Описание|
|--|--|--|--|
|kind|строка = Error|да|Описание типа ресурса|
|code|строка| да|Код ошибки определяемый приложением|
|userError|строка|да|Ошибка, которая отображается клиенту|
|systemError|строка|нет|Ошибка, имеющая смысл для разработчика|
|detail|JSON value|нет|Произвольное значение JSON описывающее детали ошибки|
</details>

<details>
<summary>Коллекция (kind=Collection)</summary>

Коллекция — это встроенный объект, представление отображающее один и более объектов других типов. Коллекция МОЖЕТ включать в себя типы определенные сервером, тип Reference и другие коллекции. Коллекция НЕ ДОЛЖНА включать в себя типы объектов Error и Page. Коллекция должна определять параметры:

|Параметр|Тип|Обязательный|Описание|
|--|--|--|--|
|kind|строка = Collection|да|Описание типа ресурса|
|contents|Array&lt;Resource object|Reference&gt;|да|Коллекция ресурсов унаследованных от Resource object, кроме Error, Page|
</details>
<details>
<summary>Страница (kind=Page)</summary>

Page — встроенный объект, страница в постраничной выборке. Если коллекция представляет всю выборку, то страница представляет лишь ее часть. Страница ДОЛЖНА содержать выборку ресурсов только из типов определенных сервером. Страниц коллекций или страниц ошибок быть не может. Каждая страница ДОЛЖНА включать параметры:

|Параметр|Тип|Обязательный|Описание|
|--|--|--|--|
|kind|строка = Page|да|Описание типа ресурса|
|pageOf|строка (URL)|да|Ссылка на коллекцию, часть которой представляет данная страница|
|total|число (целое)|да|Общее количество результатов в коллекции|
|contents|Array&lt;Resource object&gt;|да|Выборка ресурсов представляемых этой страницей|
</details>
<details>
<summary>Операция (kind=Operation)</summary>

Операция — встроенный объект, описывает long-running операцию запущенную в результате API вызова. Каждый объект Operation включает в себя параметры:

|Параметр|Тип|Обязательный|Описание|
|--|--|--|--|
|kind|строка = Operation|да|Описание типа ресурса|
|metadata|JSON value|нет|Произвольный JSON содержащий служебные метаданные|
|id|строка uuid4|да|Уникальный идентификатор операции|
|start|строка|да|timestamp времени начала операции по ISO 8601 — дата и время с указанием часового пояса|
|done|boolean|да|Если операция всё еще выполняется и возвращает false, если операция выполнена и возвращает true|
|status|строка|да|Отражает человеко-читаемое текущее состояние операции, произвольная строка|
|result|(Error|JSON)|да|В случае ошибки во время выполнения операции в result ДОЛЖЕН быть записан объект типа kind = Error. В случае успеха, в результате МОЖЕТ быть записан любой поддерживаемый спецификацией тип объекта унаследованный от Resource object, а также произвольный JSON|
</details>

</details>

___

## Use_case_description
Примеры описания логики разработки можно посмотреть в моем дипломном проекте, блок Сценарии использования

ссылка на <a href="https://docs.google.com/document/d/1IPCBv0trKXVTHWWtoMe96FaZCRDJ-uWEG2VZmhIGJAo/edit#bookmark=id.i3dr7kwihqzm">google docs</a>
___

## Functional_requirements
Пример описания функционала можно посмотреть в блоке Функциональные требования

ссылка на <a href="https://docs.google.com/document/d/1IPCBv0trKXVTHWWtoMe96FaZCRDJ-uWEG2VZmhIGJAo/edit#bookmark=id.9gdgvbxwu3cq">google docs</a>
___

## BPMN

Задание: автоматизировать систему бронирования в отеле

![Screenshot](https://github.com/nataliashved/.github-images/blob/main/bpmn.jpg?raw=true)

Задание: Моделирование бизнес-процесса оформления путёвки в туристическом агентстве
- Верхнеуровневый процесс в нотации IDEF0 
- Детализация процесса в BPMN 2.0

![Screenshot](https://github.com/nataliashved/.github-images/blob/main/bpmn_idef0.jpg?raw=true) 
___

## Activity_diagram 
Диаграмма деятельности 

Необходимо описать алгоритм работы CRM системы, в которой сотрудник поддержки заводит заявку, поступившую к нему от клиента сервиса топливных карт

![Screenshot](https://github.com/nataliashved/.github-images/blob/main/diagram_crm.jpg?raw=true)
___

## Sequence_diagram

Диаграмма последовательности sequence
![Screenshot](https://github.com/nataliashved/.github-images/blob/main/sequence.jpg?raw=true)
___

## Use_case
Диаграмма Use case из дипломного проекта
![Screenshot](https://github.com/nataliashved/.github-images/blob/main/Use%20case.jpg?raw=true)
___

## API 
Интеграции
- Определение кючевых параметров интеграции  <a href="https://docs.google.com/document/d/11UA9l0pmHD3amXFuyFQdH593jZOck3-i77AkVvsEhhw/edit?usp=sharing">google docs</a>
- Спроектировать API для кинотеатра [swagger](https://app.swaggerhub.com/apis/lianess/Iskorka2/1.0.1)

![Screenshot](https://github.com/nataliashved/.github-images/blob/main/api.jpg?raw=true)
___

## Prototypes 
Прототипирование интерфейсов
1. По заданию нужно было создать прототип главного экрана приложения по учёту расходов в Figma для iOS. Ссылка на figma <a href="https://www.figma.com/proto/fKyYFi0qrNkGeUYZLYyP6y/%D0%9F%D1%80%D0%BE%D1%82%D0%BE%D1%82%D0%B8%D0%BF?node-id=2-3&scaling=scale-down&page-id=1%3A2&starting-point-node-id=2%3A3&mode=design&t=d8PeZjRUUQvIGgCz-1" target="_blank">prototype</a> (working buttons: income, expences and calendar)
   
![Screenshot](https://github.com/nataliashved/.github-images/blob/main/figma.jpg?raw=true)

2. Прототип из дипломной работы. Эраны приложения для планирования путешествий, сделаны в <a href="https://miro.com/app/board/o9J_kz8XEt4=/?share_link_id=271714930996">Miro</a> и Photoshop
   
![Screenshot](https://github.com/nataliashved/.github-images/blob/main/prototype_diploma.jpg?raw=true)
___

## Diploma 
Дипломный проект
Задачей дипломного проекта было оформление спецификации требований на разработку программного обеспечения мобильного приложения для планирования путешествий. Были собраны требования, сделана диаграмма use case и описание вариантов использования, диаграмма классов, прототип приложения и Swagger-документация.

Ссылка на [презентацию](https://docs.google.com/presentation/d/1ApEWS3FYBY5uO59Q78Ch0QMA7xCho28i/edit?usp=sharing&ouid=115070893752402896578&rtpof=true&sd=true) и [спецификацию](https://docs.google.com/document/d/1IPCBv0trKXVTHWWtoMe96FaZCRDJ-uWEG2VZmhIGJAo/edit?usp=sharing)

![Screenshot](https://github.com/nataliashved/.github-images/blob/main/diploma_title.jpg?raw=true)

___
## DWH 
Data Warehouse
По заданию необходимо было по заданным таблицам спроектировать хранилище данных и витрину с отчетностью для онлайн университета.
В задачи входило:
- проектирование структуры хранилища данных (логической модели DWH)
- подготовка технических спецификаций
- проектирование ETL-процессов
- формулирование бизнес-правил в соответствии с принципами Data QualityOnline univercity

Core Data Layer, Data Mart Layer, ETL, технические спецификации и бизнес правила во вкладках <a href="https://docs.google.com/spreadsheets/d/17Da7IS6fAjHAVv1_yUw3HUlk2hFT_h_5kpTfDOwxJ-s/edit?usp=sharing">google sheets</a>

![Screenshot](https://github.com/nataliashved/.github-images/blob/main/dwh.jpg?raw=true)
___

## SQL
В качестве предметной области выбраны авиаперевозки по России.
![Screenshot](https://github.com/nataliashved/.github-images/blob/main/sql_diagram.jpg?raw=true)

Были ли брони, по которым не были получены посадочные талоны?

```sql 
select distinct b.book_ref, bp.boarding_no
from bookings b 
left join tickets t on t.book_ref = b.book_ref 
left join boarding_passes bp on bp.ticket_no = t.ticket_no
where bp.boarding_no is null 
order by b.book_ref
```

В каких аэропортах есть рейсы, выполняемые самолетом с максимальной дальностью перелета?

```sql
select distinct r.departure_airport_name, a.model, a."range"
from routes r
join aircrafts a on a.aircraft_code = r.aircraft_code
where "range" = (select max(a."range") from aircrafts a)
group by r.departure_airport_name, a."range", a.model 
```

Найдите процентное соотношение перелетов по типам самолетов от общего количества

```sql
select distinct a.model, (round((count(f.flight_id) over (partition by f.aircraft_code)::numeric/count(f.flight_id) over()::numeric), 2)*100)::integer||'%' "%"  
from flights f 
right join aircrafts a on a.aircraft_code = f.aircraft_code
group by a.model, f.flight_id
```

Найдите количество свободных мест для каждого рейса, их % отношение к общему количеству мест в самолете. Добавьте столбец с накопительным итогом - суммарное накопление количества вывезенных пассажиров из каждого аэропорта на каждый день. Т.е. в этом столбце должна отражаться накопительная сумма - сколько человек уже вылетело из данного аэропорта на этом или более ранних рейсах в течении дня

```sql
with cte1 as(
	select s.aircraft_code, count(s.seat_no) all_seats
	from seats s  
	group by s.aircraft_code),
cte2 as(
	select bp.flight_id, count(bp.seat_no) occupied_seats 
	from boarding_passes bp
	group by bp.flight_id)
select f.flight_no, a.airport_name, f.aircraft_code, all_seats, occupied_seats, all_seats - occupied_seats free_seats, 
(round(((all_seats - occupied_seats)::numeric/all_seats::numeric), 2)* 100)::integer||'%' "% to all_seats", f.scheduled_departure, 
sum(occupied_seats) over (partition by f.departure_airport, date_trunc('day', f.scheduled_departure) order by f.scheduled_departure) ppl_flewoutperday
from flights f 
left join cte1 c_1 on c_1.aircraft_code = f.aircraft_code 
inner join cte2 c_2 on c_2.flight_id = f.flight_id
join airports a on a.airport_code = f.departure_airport 
```
___

## BI 
Business intelligence
Задание:
Постройте дашборд на основе данных GoogleAppsStore, который бы отвечал на вопросы:
- Как выглядит распределение оценок приложений?
- Зависит ли размер приложений от категорий/жанров?
- Зависит ли качество приложений от стоимости? (за качество принимаем рейтинг).
- Показать среднюю цену платных приложений по категориям.
  
Ссылка на <a href="https://public.tableau.com/app/profile/natalia.shvedova/viz/Businessintelligence_16900433756250/Dashboard1">Tableau</a>

![Screenshot](https://github.com/nataliashved/.github-images/blob/main/tableau.jpg?raw=true)

<h1>Спасибо за просмотр моего портфолио! 😀</h1>
