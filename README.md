# Restful Booker API — тестирование

Тестовое задание на позицию QA-инженера: тест-кейсы, Postman-коллекция с автоматическими проверками и баг-репорт для REST API [Restful Booker](https://restful-booker.herokuapp.com) ([документация](https://restful-booker.herokuapp.com/apidoc/index.html)).

**Задание:** изучить документацию API, составить позитивные и негативные тест-кейсы для эндпоинтов, перенести их в Postman-коллекцию с проверками во вкладке Tests, на найденные баги оформить баг-репорт.

| Эндпоинт | Методы |
|---|---|
| `/auth` | POST |
| `/booking` | POST |
| `/booking?firstname=&lastname=&checkin=&checkout=` | GET |
| `/booking/:id` | GET, PUT, PATCH, DELETE |
| `/ping` | GET |

## Состав репозитория

| Файл | Содержимое |
|---|---|
| [`test-cases.xlsx`](test-cases.xlsx) | 85 тест-кейсов: сводка по эндпоинтам + таблица кейсов (шаги, данные, ожидаемый и фактический результат, статус, ссылка на баг) |
| [`restful-booker.postman_collection.json`](restful-booker.postman_collection.json) | Postman-коллекция: 85 запросов, по одному на тест-кейс, с проверками в Tests |
| [`restful-booker.postman_environment.json`](restful-booker.postman_environment.json) | Окружение Postman: `baseUrl`, `username`, `password` |
| [`bug-report.xlsx`](bug-report.xlsx) | Баг-репорт: 24 бага с шагами воспроизведения, ожидаемым и фактическим результатом, серьёзностью и приоритетом |

## Результаты

- **Тест-кейсы:** 85 (33 позитивных, 52 негативных) — 37 Passed, 48 Failed.
- **Postman-коллекция:** 85 запросов, 265 проверок — 203 прошли, 62 упали. Все упавшие проверки соответствуют известным багам.
- **Баги:** 24 — 2 Critical, 13 Major, 9 Minor.

| ID | Серьёзность | Описание |
|---|---|---|
| BUG-01 | Critical | PUT/PATCH /booking/:id: поле bookingid в теле запроса меняет id брони — ответ 405, бронь пропадает по исходному id |
| BUG-02 | Critical | PATCH /booking/:id: при передаче одной из дат вторая дата затирается значением "0NaN-aN-aN" |
| BUG-03 | Major | POST /booking: без обязательных полей или с неверным типом поля сервер отвечает 500 вместо 400 |
| BUG-04 | Major | POST/PUT/PATCH /booking: невалидная дата принимается и сохраняется как "0NaN-aN-aN" |
| BUG-05 | Major | POST /booking: несуществующая дата или дата в другом формате молча заменяется другой датой |
| BUG-06 | Major | POST/PUT/PATCH /booking: принимается бронь с датой выезда раньше даты заезда |
| BUG-07 | Major | POST/PUT/PATCH /booking: нечисловое значение totalprice принимается и сохраняется как null |
| BUG-08 | Major | POST /booking: дробная часть totalprice отбрасывается (150.75 → 150) |
| BUG-09 | Major | POST /booking: принимается отрицательная стоимость totalprice |
| BUG-10 | Minor | POST/PATCH /booking: нет проверки типов — строки приводятся к boolean/number, PATCH сохраняет число в firstname |
| BUG-11 | Major | POST/PATCH /booking: пустые строки и null в firstname / lastname принимаются |
| BUG-12 | Major | POST /auth: при неверных учётных данных возвращается 200 OK вместо 401 Unauthorized |
| BUG-13 | Minor | POST /auth: при отсутствии или пустых полях username / password возвращается 200 OK вместо 400 |
| BUG-14 | Major | GET /booking: фильтр checkin работает как строгое «больше», а не «больше или равно» |
| BUG-15 | Major | GET /booking: фильтр checkout работает наоборот — возвращает брони с выездом не позже заданной даты |
| BUG-16 | Major | GET /booking: невалидная дата в фильтрах checkin / checkout приводит к 500 Internal Server Error |
| BUG-17 | Minor | PUT/PATCH/DELETE /booking/:id: для несуществующей брони возвращается 405 Method Not Allowed вместо 404 |
| BUG-18 | Minor | DELETE /booking/:id: успешное удаление возвращает 201 Created |
| BUG-19 | Minor | GET /ping: health check возвращает 201 Created вместо 200 OK |
| BUG-20 | Minor | Ответы в формате XML отдаются с Content-Type: text/html вместо application/xml |
| BUG-21 | Major | Формат ответа application/x-www-form-urlencoded не работает: 200 OK и текст ошибки «formurlencoded is not a function» |
| BUG-22 | Minor | Неподдерживаемый формат в Accept возвращает 418 I'm a Teapot вместо 406 Not Acceptable |
| BUG-23 | Minor | POST /booking: неподдерживаемый Content-Type приводит к 500 Internal Server Error вместо 415 |
| BUG-24 | Minor | PUT /booking/:id: необязательное поле additionalneeds не удаляется, если его нет в теле запроса |

## Как запустить коллекцию

### Postman

1. **Import** → перетащить оба JSON-файла: коллекцию и окружение.
2. В правом верхнем углу выбрать окружение **Restful Booker**.
3. Вся коллекция: - у коллекции → **Run collection** → **Run**. Runner покажет результат каждой проверки.
4. Отдельный запрос: открыть → **Send** → вкладка **Test Results** в панели ответа.

Код проверок — во вкладке запроса **Scripts → Post-response**.

### Newman

```bash
npx newman run restful-booker.postman_collection.json -e restful-booker.postman_environment.json
```

Прогон занимает около 2–3 минут. Код выхода ненулевой, потому что проверки на известных багах падают.

## Как устроены проверки

- **Тесты проверяют корректное поведение**, а не подстраиваются под фактическое. Ожидаемый результат взят из документации API и семантики HTTP (например, неверный пароль → `401`, а не `200`).
- **Упавшая проверка = известный баг.** Имя таких проверок начинается с номера бага: `[BUG-12] Код ответа 401`. Если баг исправят, проверка начнёт проходить без изменений коллекции.
- **Запросы независимы.** Pre-request скрипты папок перед каждым запросом создают свежую бронь и получают токен, поэтому любой запрос можно отправить отдельно, в любом порядке.
- **Что проверяется:** код ответа, `Content-Type`, JSON-схема ответа, значения полей; после изменяющих запросов — повторный `GET`, что данные действительно сохранились (или не изменились при ошибке); время ответа < 3 с для всех запросов.
- **Имена запросов совпадают с ID тест-кейсов** (`TC-PUT-03 …`), в описании каждого запроса — ожидаемый результат и номер бага.
