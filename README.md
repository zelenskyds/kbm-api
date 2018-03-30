## Заявка (Application)

```JS
{
    "id": <integer>, // id заявки в БД
    "status": <string>, // статус заявки, см. ниже
    "paid": <boolean>, // оплачена ли заявка
    "rate": <integer>, // стоимость этой заявки
    "createdAt": <string>, // дата создания в стандартном формате JS, например `Wed Mar 25 2015 03:00:00 GMT+0300 (MSK)`
    "ownerId": <integer>, // id пользователя-создателя заявки
    "policyId": <integer>, // id полиса - в нем содержится информация о водителе
}
```

`status` - текущий статус заявки, может быть:
 - `at work` - заявка обрабатывается
 - `done` - заявка успешно отработана
 - `canceled` - зявка отработана, но `КБМ` не изменился, или изменился на `0.05`
 - `waiting` - зявка ожидает отправки в работу, при таком статусе ее можно редактировать
 - `stopped` - заявка приостановлена, по причине отрицательного баланса. Можно редактировать


 ## Полис (Policy)

 ```JS
 {
     "id": <integer>, // id полиса в БД
     "firstName": <string>, // имя
     "lastName": <string>, // фамилия
     "patronymic": <string>, // отчество
     "birthday": <string>, // дата рождения, `гггг-мм-ддT00:00:00.000Z`
     "kbm": <integer>, // коэффициент бонус-малус, текущий
     "oldKbm": <integer>, // коэффициент бонус-малус на момент создания заявки
     "oldName": <string>, // старое ФИО
     "driverDoc": <string>, // водительское удостоверение
     "oldDriverDoc": <string>, // старое водительское удостоверение
 }
 ```

## Получение токена

POST: https://kbm-partner.ru/api/sessions

__Заголовки запроса__:
```
Content-Type: application/json
```

__Тело запроса:__
```JS
{
  name: <string>, // email пользователя
  password: <string> // пароль пользователя
}
```

__Ответ:__
```JS
{
  secret: <string>, // токен
  userId: <int> // id пользователя в БД
}
```

## Проверка токена

GET: https://kbm-partner.ru/api/sessions/:token

`:token` - токен

__Заголовки запроса__:
```
Content-Type: application/json
```

__Ответ:__
```JS
{
  userId: <int> // id пользователя в БД
}
```


## Создание заявки

POST https://kbm-partner.ru/api/applications

__Заголовки запроса__:
```
Content-Type: application/json
X-Auth: <SESSION_TOKEN>
```

__Тело запроса:__
```JS
{
  "firstName": <string>, // имя
  "lastName": <string>, // фамилия
  "patronymic": <string>, // отчество, опционально
  "birthday": <string>, // дата рождения, дд.мм.гггг
  "kbm": <number>, // коэффициент бонус-малус
  "driverDoc": <string>, // водительское удостоверение
  "oldDriverDoc": <string>, // предыдущее водительское удостоверение, опционально
  "oldName": <string>, // старые ФИО, опционально
}
```

__Ответ:__
```JS
{
  "applicationId": <integer> // id заявки в БД
}
```

## Получение заявки

GET https://kbm-partner.ru/api/applications/:id

`:id` - id заявки в БД

__Заголовки запроса__:
```
Content-Type: application/json
X-Auth: <SESSION_TOKEN>
```

__Ответ:__
```JS
<Application> // см. структуру заявки
```
в ответ включена информация о полисе в поле `policy` (см. <Policy>)

## Получение заявок

GET https://kbm-partner.ru/api/applications?offset=__int__&limit=__int__&status=__array_of_string__&embedded=__boolean__

`offset` - сколько заявок от начала пропустить (смещение). Опционально  
`limit` - ограничение количества заявок. Опционально  
`status` - массив статусов заявок, например `status=[done, canceled]`. Опционально  
`embedded` - если true, в каждую `<Application>` будет добавлено поле `policy`, которое будет иметь структуру `<Policy>`

__Заголовки запроса__:
```
Content-Type: application/json
X-Auth: <SESSION_TOKEN>
```

__Ответ:__

```JS
{
  "applications": <array of <Application>>, // массив заявок
  "limit": <integer>, // см. `limit`
  "offset": <integer>, // см. `offset`
  "left": <integer>, // <количество заявок> - <offset>
}
```
