# Тестирование

Все публичные методы должны быть покрыты модульными тестами. 
Как говорится, "доверяй, но проверяй" - лучше один модульный тест на метод, чем тысяча заверений в его работоспособности. 
В требовании писать модульные тесты мы исходим из экономической эффективности. 
Написание теста занимает 10 минут, а обнаружение бага тестировщиком, заведение задачи и последующее устранение бага разработчиком может занимать от часа и более. 
Поэтому мы пишем модульные тесты.

## Модульное тестирование

### Четыре этапа тестирования

Каждый тест состоит из четырёх этапов:

1. Подготовка данных.
2. Тестирование проверяемой функциональности.
3. Проверка полученных результатов.
4. Возврат приложения в первоначальное состояние.

Пример теста, со всеми четырьмя этапами тестирования функциональности:

```kotlin
@Test
fun delete() {
    //prepare
    entityGenerator.insertCalculationType(domain.id!!) //подготовка тестовых данных
        .also {
            //when
            calculationTypeService.delete(it.id!!) //тестирование проверяемой функциональности
            //then
            assertThrows(NoSuchElementException::class.java) { calculationTypeService.get(it.id!!) } //проверка полученных результатов
        }
    //tear down
    entityGenerator.deleteCalculationType(domain.id)
}
```

### Profile или Mockito?

В тех случаях, когда требуется описать нестандартное поведение какого-то сервиса, мы создаём бины-наследники основного интерфейса с нестандартным поведением. 
Например, у нас существует интерфейс `UserService`. 
Основной бин, в котором содержится ожидаемая логика поведения - `UserServiceImpl`. 
Бин, в котором описана негативная логика (например, выбрасываются исключения), будет называться `UserServiceNegative`, при этом, поскольку бин тестовый, он содержится в тестовом модуле. 
Инициализация тех или иных бинов управляется через профили. 
Использование библиотеки Mockito не приветствуется, поскольку при таком подходе поведение одного бина описывается в другом.

## Пользовательское тестирование

По взаимодействию со специалистами QA, мы следуем следующим правилам.

В задаче на исправление бага мы ожидаем полный запрос и полный ответ в виде текстовых файлов.

### Пример запроса

```shell
curl --location 'http://bpmu-ingress-bpmu-backend-bpmu-back-dev.apps.dap.devpub-02.solution.sbt/bpmu/graphql' \
--header 'Content-Type: application/json' \
--data '{
    "query": "query getProcessByDefinitionId($where: ProcessDefinitionArgument, $orderBy: ProcessDefinitionOrderBy, $pagination: Pagination) { ProcessDefinitions(where: $where, orderBy: $orderBy, pagination: $pagination) { processDefinitions { id process { id module { id endpoint version podName } state createdAt updatedAt operations { id type username createdAt updatedAt moduleIds extendedData { totalCount currentCount processId } } } name createdAt updatedAt version uploadVersion businessFamily ownerRole pinned active schema retryPolicyModels { id name description retryPolicies { id description exceptions errorExpressions exceptionMessages errorCodes retryStrategy linearCount linearTimeout intervals } } } aggregation { totalCount } } }",
    "variables": {
        "orderBy": {
        }
    },
    "operationName": "getProcessByDefinitionId"
}'
```

Запрос должен быть в человекочитаемом формате curl без артефактов форматирования, в отдельном файле.

### Пример ответа

```json
{
    "data": {
        "ProcessDefinitions": {
            "processDefinitions": [],
            "aggregation": {
                "totalCount": 0
            }
        }
    }
}
```

Ответ должен содержать HTTP-статус и тело ответа, отдельным файлом.