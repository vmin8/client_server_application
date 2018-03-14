# Приложение клиент - сервер для работы с метриками

Данное приложение было разработано в рамках [курса по языку Python](https://www.coursera.org/learn/programming-in-python) от МФТИ и Mail.ru Group на платформе Coursera. 

Основная задача курса заключалась в том, чтобы разобраться в объектной модели языка `Python` и применить полученные знания для создания высоконагруженных приложений. Примером таких приложений могут являться различные веб-сервисы, предоставляющие доступ к интересующей информации сотням и тысячам пользователей одновременно. 

Для организации сетевого взаимодействия таких систем был рассмотрен механизм [сокетов](https://docs.python.org/3/howto/sockets.html), для поддержки возможности параллельной обработки большого количества запросов - основные концепции асинхронного программирования, в частности, особое внимание было уделено библиотеке [`asyncio`](https://docs.python.org/3/library/asyncio.html).

## Постановка задачи

Итоговым заданием данного курса являлось создание двух модулей:
1. Клиента для отправки метрик на сервер и запросов интересующих метрик от сервера.
2. Сервера, получающего запросы от клиентов и возвращающих результаты согласно протоколу. 

### Клиент для отправки метрик

1. Клиент устанавливает соединение с сервером при помощи механизма сокетов с использованием адресной пары (хост, порт).
2. Клиент использует методы `put(key, value, timestamp)` и `get(key)` для отправки данных на сервер и получения необходимой информации от сервера соответственно.

#### Метод `put`:
```
def put(self, key, value, timestamp):
    ...
```
Данный метод получает от пользователя название метрики (`key`), её значение (`value`), полученное в момент времени `timestamp`. 
Внутри метода данные преобразуются к формату запроса, ожидаемого сервером:
```
message = "put <key> <value> <timestamp>\n"
```
Затем запрос отправляется на сервер с помощью соединения, установленного при инициализации объекта класса `Client()`. 

Метод `put` не возвращает никаких значений и выбрасывает исключения в зависимости от произошедших в ходе выполнения ошибок.

#### Метод `get`:
```
def get(self, key):
    ...
```
Данный метод получает от пользователя название метрики (`key`) или значение `"*"`, формирует запрос к серверу следующего вида:
```
message = "get <key>\n"
```

По аналогии с методом `put` запрос отправляется на сервер с помощью сохраненного соединения в объекте класса `Client()`. 

Ответом от сервера на запрос является строка, содержащая информацию обо всех метриках, имеющих название `<key>`, либо всю информацию, содержащуюся на сервере в случае запроса с параметром `<key> == "*"`.

Метод `get` преобразует полученную строку в словарь, хранящий информацию о метриках в следующем формате:
```
{key : [(timestamp_1, value_1), \dots, (timestamp_n, value_n)]}
```
