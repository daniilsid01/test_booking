# Тестовое задание: Restful-Booker API

Проект по функциональному тестированию REST-API **Restful-Booker**
(<https://restful-booker.herokuapp.com/apidoc/index.html>).

## Состав проекта

| Артефакт | Файл | Описание |
|---|---|---|
| Postman-коллекция | `postman/Restful-Booker_Test-Suite.postman_collection.json` | 52 запроса в 8 папках: все эндпоинты из задания (`GET /ping`, `POST /auth`, `POST /booking`, `GET /booking` + фильтры, `GET/PUT/PATCH/DELETE /booking/:id`) с проверками во вкладке Tests. Экспортирована в формате коллекции v2.1.0 |
| Тест-кейсы | `test_cases/Test-Cases_Restful-Booker.xlsx` | 53 тест-кейса (позитивные/негативные) с предусловиями, шагами, данными, ожидаемым и фактическим результатом, статусом и привязкой к багам |
| Баг-репорты | `bug_reports/Bug-Reports_Restful-Booker.xlsx` | 9 дефектов с шагами воспроизведения, ожидаемым/фактическим поведением, данными запросов/ответов, severity/priority |

## Окружение и даты

- База: `https://restful-booker.herokuapp.com` — общий публичный стенд (Heroku, free-tier).
- Дата тестирования: **10.09.2026 (UTC)**. Инструменты: curl, Postman/Newman.
- Стенд **периодически сбрасывает/дополняет данные и перезапускается**: токены хранятся в памяти и после рестарта могут стать невалидными (ответ `403`), а созданные брони могут исчезать. Поэтому все «мутационные» запросы в коллекции **самодостаточны**: в pre-request они создают собственную бронь-фикстуру и при необходимости получают токен. Если во время прогона стенд перезапустился, достаточно повторить прогон.

## Как запустить коллекцию

**Postman (UI):**
1. `Import` → выбрать `postman/Restful-Booker_Test-Suite.postman_collection.json`.
2. Переменные коллекции уже заполнены (`base_url` = `https://restful-booker.herokuapp.com`).
3. `Run collection` (все папки, порядок сверху вниз).

**Newman (CLI):**

```bash
newman run postman/Restful-Booker_Test-Suite.postman_collection.json
```

## Результат контрольного прогона (Newman)

Все 23 упавшие проверки соответствуют задокументированным дефектам `BUG-01…BUG-09`
(см. `bug_reports/Bug-Reports_Restful-Booker.xlsx`); запросы с пометкой
`[известный дефект BUG-XX]` в названии ожидаемо завершаются **FAIL до исправления дефекта**.
Остальные проверки проходят. Если во время прогона стенд перезапустился (все защищённые
запросы начинают падать с `403`), повторите прогон.

## Краткая карта найденных дефектов

| ID | Эндпоинт | Суть |
|---|---|---|
| BUG-01 | `GET /ping` | Health-check отвечает `201 Created` вместо `200 OK` |
| BUG-02 | `POST /auth` | Неверные/неполные учётные данные → `200 OK {"reason":"Bad credentials"}` вместо `401/400` |
| BUG-03 | `POST /booking` | Нет валидации обязательных полей и типов: пустое/неполное/некорректно типизированное тело → `500 Internal Server Error` вместо `400` |
| BUG-04 | `POST /booking`, `PATCH /booking/:id` | Невалидные даты/типы принимаются и портят данные: `"notadate"` сохраняется как `"0NaN-aN-aN"`, `totalprice:"abc"` — как `null`, допускается `checkout < checkin` |
| BUG-05 | `GET /booking?checkin&checkout` | Off-by-one в фильтре по датам: бронь с `checkin`, равным границе запроса, не находится (хотя документация обещает «greater than or equal»); в выдачу попадают брони с повреждёнными датами |
| BUG-06 | `PUT/PATCH/DELETE /booking/:id` | Операции с несуществующей (в т.ч. уже удалённой) бронью → `405 Method Not Allowed` вместо `404 Not Found` |
| BUG-07 | `DELETE /booking/:id` | Успешное удаление возвращает `201 Created` — семантически неверный код для DELETE |
| BUG-08 | `GET /booking/:id` | Content negotiation: `Accept: text/plain` → `418 I'm a teapot` вместо `406`; при `Accept: application/xml` отдаётся XML с `Content-Type: text/html` |
| BUG-09 | `POST /auth` | Документированный обязательный заголовок `Content-Type: application/json` не обязателен — принимается также `x-www-form-urlencoded` (расхождение документации и реализации, низкая значимость) |

Детали (шаги, ожидание/факт, сырые ответы, severity/priority) — в файле
`bug_reports/Bug-Reports_Restful-Booker.xlsx`.

## Структура

```
.
├── postman/
│   └── Restful-Booker_Test-Suite.postman_collection.json   # коллекция (экспорт v2.1.0)
├── test_cases/
│   └── Test-Cases_Restful-Booker.xlsx                      # 53 тест-кейса
├── bug_reports/
│   └── Bug-Reports_Restful-Booker.xlsx                     # 9 баг-репортов
└── README.md
```
