# Обработка исключений

Для любой задачи может существовать два типа сценариев - один позитивный и `n` негативных. 
Мы стараемся избегать выполнения нескольких позитивных сценариев в приложении, равно как ветвлений в целом. 
Позитивный сценарий выполнения задачи выполняется в основной части кода и в блоке try там, где это необходимо. 
Негативные сценарии обрабатываются в блоках catch и в `ExceptionHandler`. 
Обработка исключения в `ExceptionHandler` предпочтительнее обработки в блоке `catch`.

При обработке исключений мы стараемся оперировать исключениями, уже существующими в `java.lang` и используемых библиотеках. 
Добавление кастомных исключений не приветствуется.

Все исключительные ситуации для задач, подразумевающих возврат ответа клиенту, обрабатываются в `ExceptionHandler` и связаны с определённым HTTP-статусом.

(статус-коды ниже приведены для примера и являются предметом дискуссии)

* `500 (Internal Server Error)` : логическая ошибка на стороне приложения.
* `400 (Bad Request)`           : ошибка запроса со стороны клиента.