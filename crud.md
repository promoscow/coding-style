# CRUD-операции

Мы придерживаемся некоторых правил для CRUD-операций, в части получаемых и возвращаемых данных.

## Сохранение данных

При сохранении данных, мы хотим точно знать, в каком именно виде данные сохранил внешний интерфейс.
Поэтому, при сохранении в слое DAO, мы получаем данные из внешнего интерфейса "как есть", мапим в бизнес-модель и возвращаем бизнес-модель.
Возвращаемого идентификатора мало - помимо идентификатора, внешний интерфейс (например, база данных) может генерировать дополнительные свойства, критичные для бизнес-модели.

В названии функции мы как можно более точно указываем, что именно она делает.
Например, сервисная функция сохранения данных может сохранить данные в базе данных, а потом дообогатить их из других сервисов.
Такую функцию лучше назвать обобщённым именем `save`.
Если функция, помимо основной бизнес-сущности, попутно сохраняет вложенные бизнес-сущности (мы стараемся так не делать, но для примера подойдёт), такую функцию можно назвать ещё более общим именем - `create`. 
Если репозиторная функция только записывает сущность в базу, её лучше назвать наиболее узким именем: `insert`.

Пример объявления функции `insert`:

```kotlin
fun insert(settlement: Settlement): Settlement
```

На примере выше, мы принимаем бизнес-модель и возвращаем бизнес-модель целиком.

Пример инициализации функции `insert`:

```kotlin
override fun insert(settlement: Settlement): Settlement = transaction {
    SettlementEntity
        .insert { settlement.toInsertStatement(it) }
        .resultedValues
        ?.firstOrNull()
        ?.let { it[SettlementEntity.id].value }
        ?.let { find(it) ?: throw NoSuchElementException("Settlement not found by resulted id: $it.") }
        ?: throw NoSuchElementException("Error inserting settlement ${settlement.toJson()}. Result is null.")
    }
```

Как мы видим, функция возвращает именно те значения, которые были сохранены в базу.

## Обновление данных

При обновлении данных, приветствуется возвращение обновлённой сущности, с получением её из базы данных.
В большинстве случаев, для этого придётся лишний раз сходить в базу данных.

Пример реализации функции обновления данных:

```kotlin
override fun update(payslipDocument: PayslipDocument): PayslipDocument = 
    transaction {
        PayslipDocumentEntity
            .update({ PayslipDocumentEntity.id eq payslipDocument.id }) { it.enrichWith(payslipDocument) }
            .let { 
                find(payslipDocument.id!!)
                    ?: throw Exception("Updated payslip document not found by id: ${payslipDocument.id}")
            }
    }
```

Если операция получения данных слишком затратная, допускается не возвращать обновлённую сущность.

## Получение данных

При получении данных мы используем именование `get`, если мы планируем получить non-nullable ответ, и `find` - если планируем получить nullable.

Если поиск осуществляется по основному идентификатору сущности (`id`), не нужно пытаться раскрыть название полнее, чем необходимо.
Не нужно писать `get(documentId: UUID): Document`, если `documentId` - это и есть основной `id` документа.
Дополнительное описание в наименовании создаст ложное впечатление, что у сущности `Document` есть поле `documentId` отдельно от поля `id`.

## Удаление данных

При удалении данных мы обычно не возвращаем значение.
Максимум, что можно вернуть - boolean флаг о том, что удаление произошло успешно.
Признаком успешности удаления сущности служит количество удалённых строк, совпадающее с планируемым количеством.

Пример такой функции:

```kotlin
override fun delete(id: UUID): Boolean =
    transaction {
        DocumentEntity
            .deleteWhere { DocumentEntity.id eq id }
            .also { deletedCount ->
                if (deletedCount != rowsToDeleteById) {
                    throw Exception("Error deleting document by id: $id. Deleted $deletedCount rows, expected: $rowsToDeleteById")
                }
            }
    }
```