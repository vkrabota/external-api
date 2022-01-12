# VK Работа External API

Внешнее API VK Работа предоставляет возможности для автоматизированного добавления новых вакансий, управления статусами вакансий, получения списка откликов, управления статусами откликов, поиска и покупки контактов.

Для подключения возможности работы через API напишите нам на support@vkrabota.ru.

## Содержание

* [Аббревиатуры](#аббревиатуры)
* [Основные схемы данных](#основные-схемы-данных)
  * [Job](#job)
  * [Category](#category)
  * [Profession](#profession)
  * [Application](#application)
  * [Candidate](#candidate)
  * [Company](#company)
  * [Vas Transaction](#vas-transaction)
  * [Paid Action](#paid-action)
  * [Region](#region)
  * [City](#city)
* [Аутентификация в API VK Работа](#аутентификация-в-api-vk-работа)
  * [Использование сессионных токенов](#использование-сессионных-токенов)
  * [Получение сессионного токена](#получение-сессионного-токена)
* [Вакансии](#вакансии)
  * [Получение списка вакансий](#получение-списка-вакансий)
  * [Создание вакансии](#создание-вакансии)
  * [Изменение вакансии](#изменение-вакансии)
  * [Получение информации о конкретной вакансии](#получение-информации-о-конкретной-вакансии)
  * [Закрытие вакансии](#закрытие-вакансии)
  * [Продление вакансии](#продление-вакансии)
* [Получение списка категорий профессий](#получение-списка-категорий-профессий)
* [Получение списка профессий](#получение-списка-профессий)
* [Получение списка регионов](#получение-списка-регионов)
* [Получение списка городов](#получение-списка-городов)
* [Получение списка гражданств](#получение-списка-гражданств)
* [Отклики](#отклики)
  * [Получение списка откликов](#получение-списка-откликов)
  * [Изменение статуса отклика](#изменение-статуса-отклика)
* [Баланс компании и VAS'ы](#баланс-компании-и-vasы)
  * [Получение состояния баланса компании](#получение-состояния-баланса-компании)
  * [Получение списка транзакций VAS'ов](#получение-списка-транзакций-vasов)
  * [Применение VAS'ов](#применение-vasов)
* [Поиск и покупка контактов из базы резюме](#поиск-и-покупка-контактов-из-базы-резюме)
  * [Поиск кандидатов](#поиск-кандидатов)
  * [Покупка контакта](#покупка-контакта)
  * [Просмотр контакта](#просмотр-контакта)
* [Фоновое изображение](#фоновое-изображение)
  * [Загрузка фонового изображения](#загрузка-фонового-изображения)
  * [Удаление фонового изображения](#удаление-фонового-изображения)

## Аббревиатуры
`VAS` - Value Added Service. Услуга, которую мы предоставляем. VAS'ы бывают:
1) размещение вакансии. Код: `job_publication` для обычной публикации, `super_job_publish` для продвинутой публикации, `ultra_job_publish` для премиум публикации
2) поднятие вакансии в поиске. Код `job_refresh`.
3) выделение вакансии на 24 часа. Код `job_highlight`.
4) вакансия дня на 1/3/7 дней. Код 7 дней - `elevation_plus`, 3 дней - `elevation_3d`, 1 день - `elevation_1d`.
5) просмотр контакта кандидата. Код `contact_view`.

`OTVAS` - One Time VAS. Единоразово покупаемые VAS'ы, которые начисляются на баланс компании. Имеют срок жизни в 30 дней с момента последней покупки OTVAS'ов.

## Основные схемы данных

В данном API реализовано несколько сущностей, которые могут быть описаны следующими схемами:

#### Job

Модель `Job` служит для хранения информации о вакансии и в общем случае имеет следующие атрибуты:

* `id` - первичный ключ, идентификатор вакансии в API VK Работа
* `status` - статус вакансии, константа, строка, принимающее только одно значение из словаря: `active` - вакансия открыта, `closed` - вакансия закрыта, `expired` - срок действия вакансии закончился
* `title` - обязательный атрибут, заголовок вакансии, строка, обычно - название компании
* `description` - обязательный атрибут, описание вакансии, текстовое поле, не должно содержать более 200 символов, не допускаются номера телефонов, адреса сайтов и прочая информация, которая может быть рассмотрена как контактная информация
* `lat` - обязательный атрибут, широта географической точки вакансии - вещественное число, принимающее значение от -90 до 90
* `long` - обязательный атрибут, долгота географической точки вакансии - вещественное число, принимающее значение от -180 до 180
* `address` - обязательный атрибут, строка - полный адрес вакансии
* `profession_id` - обязательный атрибут, идентификатор профессии из справочника профессий
* `salary_from` - необязательный атрибут, целое число, нижняя планка заработной платы профессии
* `salary_to` - необязательный атрибут, целое число, верхняя планка заработной платы профессии
* `salary_period` - необязательный атрибут, строка, устанавливает тип оплаты и принимает только одно значение из словаря: `hourly` - суммы `salary_from/salary_to` оплата за час, `daily` - оплата за день/смену, `monthly` - месячная заработная плата
* `autoreply_text` - текст автоответа
* `has_autoreply` - включён ли автоответ
* `contact_phone` - контактный телефон конкретной вакансии в формате `7XXXXXXXXXX`
* `expires_at` - дата истечения времени публикации вакансии
* `auto_renew` - флаг автоматического продления вакансии
* `remote` - удаленная работа
* `current_publication_type` - текущий тип публикации. Бывает: `not_published` - не опубликована, `job_publish` - обычная публикация, `super_job_publish` - продвинутая публикация, `ultra_job_publish` - премиум публикация

#### Category

Модель `Category` служит для хранения справочника категорий вакансий в API VK Работа и имеет следующие атрибуты:

* `id` - первичный ключ, идентификатор категории профессий в API VK Работа
* `name` - название категории профессий, строка
* `transliterated_name` - транслитерация названия категории профессий, строка
* `popular` - признак популярности категории профессий, тип boolean
* `sorting_value` - порядок сортировки категории профессий, число
* `salary_to` - максимальное значение по зарплате в месяц для категории профессий в рублях, число
* `icon_image_url` - путь до иконки категории профессий, строка
* `alternative_icon_image_url` - путь до альтернативной иконки категории профессий, строка

#### Profession

Модель `Profession` служит для хранения справочника вакансий в API VK Работа и имеет следующие атрибуты:

* `id` - первичный ключ, идентификатор профессии в API VK Работа
* `title` - обязательный атрибут, название профессии, строка

#### Application

Модель `Application` служит для хранения информации об отклике, устанавливает связь между вакансией (модель `Job`) и кандидатом (модель `Candidate`) и имеет следующие атрибуты:

* `id` - первичный ключ, идентификатор отклика в API VK Работа
* `job_id` - обязательный атрибут, идентификатор вакансии
* `candidate_id` - обязательный атрибут, идентификатор кандидата
* `recruiter_id` - обязательный атрибут, идентификатор работодателя
* `status` - обязательный атрибут, строка, устанавливает статус отклика и может принимать только одно значение из словаря: `unread` - непрочитанный отклик, `declined` - кандидату отказано, `read` - отклик в работе, `selected` - кандидат выбран для дальнейшей обработки, `improper` - кандидат не подошел, `rejected` - кандидату отказано, `invited` - кандидат приглашен на собеседование, `reserved` - кандидат зарезервирован для дальнейшего прохождения собеседования, `missed` - кандидат не пришел на собеседование и получил отказ, `closed` - служебный статус для откликов на закрытые вакансии, которые были не обработаны
* `created_at` - обязательный атрибут, дата и время создания отклика

В каждый момент времени один кандидат может иметь только один отклик на одну вакансию.

#### Candidate

Модель `Candidate` служит для хранения информации о кандидате и имеет следующие атрибуты:

* `id` - идентификатор пользователя в системе
* `first_name` - обязательный атрибут, строка, имя кандидата
* `last_name` - обязательный атрибут, строка, фамилия кандидата
* `phone` - обязательный атрибут, строка, телефон кандидата
* `description` - обязательный атрибут, текстовое описание информации кандидата о себе
* `lat` - обязательный атрибут, широта географической точки местоположения кандидата - вещественное число, принимающее значение от -90 до 90
* `long` - обязательный атрибут, долгота географической точки местоположения кандидата - вещественное число, принимающее значение от -180 до 180
* `address` - обязательный атрибут, строка - полный адрес местоположения кандидата
* `education` - необязательный атрибут, текстовая информация об образовании кандидата
* `languages` - необязательный атрибут, текстовая информация о владении языками кандидата
* `last_online_time` - необязательный атрибут, время последнего входа в приложение VK Работа
* `created_at` - обязательный атрибут, дата и время регистрации кандидата
* `updated_at` - обязательный атрибут, дата и время последнего обновления кандидатом информации о себе
* `email` - необязательный атрибут, строка, контактный email кандидата
* `birthday` - необязательный атрибут, дата рождения кандидата
* `driving_license_categories` - необязательный атрибут, массив строк с информации об открытых категориях водительских прав кандидата
* `medical_record` - необязательный атрибут, флаг, сигнализирующий о наличии медицинской книжки у кандидата
* `active_candidate` - обязательный атрибут, флаг, сигнализирующий о том, что кандидат действительно рассматривает предложения и активен в системе
* `accept_calls` - обязательный атрибут, флаг, сигнализирующий о желании кандидата получать входящие звонки от работодателей
* `avatar` - необязательный атрибут, строка, ссылка на фотографию кандидата
* `background` - необязательный атрибут, строка, ссылка на дополнительную фотографию кандидата
* `experiences` - необязательный атрибут, массив с информацией о прошлом опыте работы, имеет следующую структуру:
    * `title` - обязательный атрибут, строка, название места работы
    * `description` - обязательный атрибут, текстовое описание рабочих обязанностей
    * `duration` - необязательный атрибут, число месяцев работы на данном месте
    * `current` - обязательный атрибут, флаг, сигнализирующий, что данное место работы является текущим
* `nationality` - необязательный атрибут, строка, информация о гражданстве кандидата (всегда true при покупке)
* `contact_bought` - необязательный атрибут, флаг, сигнализирующий о списании средств за покупку контакта
* `already_contacted` - необязательный атрибут, флаг, сигнализирующий о ранее приобретенном контакте (всегда false при покупке)
* `spent_counts` - необязательный атрибут, объект с информацией о лимитах, имеет следующую структуру:
    * `count_for_packets` - обязательный атрибут, лимиты по пакетам
    * `count_for_subscription` - обязательный атрибут, лимиты по подпискам (deprecated)
    * `count_for_company` - обязательный атрибут, лимиты по компаниям (deprecated)
    * `count_for_bonus` - обязательный атрибут, лимиты по бонусам подпискок (deprecated)
    * `count_for_services` - обязательный атрибут, лимиты по сервисам
* `packet_used` - необязательный атрибут, флаг, сигнализирующий об использовании данных с пакета (всегда true при покупке контакта)
* `contact_used` - необязательный атрибут, флаг, сигнализирующий о покупке данных с пакета

#### Company

Модель `Company` служит для хранения информации о компании, балансе денежных средств и OTVAS'ов.
* `id` - идентификатор компании в системе
* `otvas_jobs_limit` - количество доступных размещений вакансий
* `otvas_highlights_limit` - количество доступных выделений вакансий
* `otvas_refreshes_limit` - количество доступных поднятий в поиске
* `otvas_elevations_plus_limit` - количество доступных вакансий дня на 7 дней
* `otvas_elevation_3d_limit` - количество доступных вакансий дня на 3 дня
* `otvas_elevation_1d_limit` - количество доступных вакансий дня на 1 день
* `views_limit` - количество доступных просмотров контактов пользователей
* `balance` - баланс кошелька компании в копейках
* `wallet_id` - номер кошелька компании
* `otvas_expires_at` - дата истечения срока жизни OTVAS'ов

#### Vas Transaction

Модель `Vas Transaction` служит для хранения данных об изменениях баланса VAS'ов.
* `created_at` - дата создания транзакции
* `updated_at` - дата изменения транзакции
* `subscriptable_type` - тип VAS'а, который тратится(MVAS, OTVAS, BVAS)
* `creator` - автор транзакции
* `applied_changes` - массив совершенных изменений

#### Paid Action

Модель `Paid Action` является служебной для возвращения данных о только что примененных/использованных VAS'ах.
* `error_message` - текст сообщения об ошибке(если ошибки нет - `null`)
* `status` - статус применения VAS'a. (`success` | `failed`)
* `origin` - исходный запрос
    * `action` - код VAS'a
    * `count` - при применении VAS'a нормализуется до 1
    * `job_id` - id вакансии, к которой применяется VAS.
* `job` - вакансия, к которой применялся VAS.

Здесь и далее необязательные атрибуты могут принимать значение `null`, быть пустыми массивами, а также не требоваться для создания новых объектов, обязательные атрибуты всегда имеют значение не `null` и должны передаваться для создания новых объектов.

#### Region

Модель `Region` содержит данные о регионе вакансии.
* `fias_id` — guid региона согласно [ФИАС](https://fias.nalog.ru/)
* `name`— название региона

#### City

Модель `City` содержит данные о городе вакансии.
* `id` — id города согласно базе VK Работа
* `name`— название города

## Аутентификация в API VK Работа

### Использование сессионных токенов

В качестве схемы аутентификации в API VK Работа используется JSON Web Tokens (JWT). Для доступа к ресурсам API VK Работа в запросе необходимо отправить заголовок следующего вида:

```
Authorization: Worki <token>
```

Где `token` - полученный токен сессии.

В случае успешной аутентификации будет произведено запрашиваемое действие. В случае невалидного или отсутствующего токена методы API будут отдавать:

```
$ curl -i https://api.iconjob.co/api/external/v1/jobs

HTTP/1.1 401 Unauthorized
Content-Type: application/json; charset=utf-8

{"errors":[{"detail":"Invalid JWT Token"}]}
```

### Получение сессионного токена

Для получения сессионного токена необходимо отправить POST-запрос по адресу `/api/external/v1/session` и передать параметры `login` и `password`.

При отсутствии параметров `login` и `password` будет возвращена следующая ошибка:

```
$ curl -i -XPOST https://api.iconjob.co/api/external/v1/session

HTTP/1.1 422 Unprocessable Entity
Content-Type: application/json; charset=utf-8

{"errors":[{"detail":"Invalid Login"}]}
```

В случае некорректных значений параметров `login` и `password` будет возвращена следующая ошибка:

```
$ curl -i -XPOST https://api.iconjob.co/api/external/v1/session -d 'login=test&password=wrong'

HTTP/1.1 401 Unauthorized
Content-Type: application/json; charset=utf-8

{"errors":[{"detail":"Invalid Auth params"}]}
```

В случае корректных значений `login` и `password` будет возвращен токен в следующем виде:

```
$ curl -i -XPOST https://api.iconjob.co/api/external/v1/session -d 'login=test&password=correct'

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VyX3R5cGUiOiJBcGlVc2VyIiwiZXhwIjoxNTMwMjI5NDI5fQ.DOmtBLljWXtk7bcvER_NCz-X1tmNoyLxWyfgTTukQbU"}
```

## Вакансии

### Получение списка вакансий

Для получения списка вакансий необходимо отправить GET-запрос на `/api/external/v1/jobs`.

Опциональные параметры:
* `page` - параметры для пагинации:
    * `number` - номер страницы
    * `size` - количество элементов на странице (не более 50)

Результат будет представлен в следующем виде (JSON API-ответ сжат и отформатирован для наглядности):

```
$ curl -i -H 'Authorization: Worki <JWT Token>' https://api.iconjob.co/api/external/v1/jobs

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "data": [
        {
            "id": "254",
            "type": "job",
            "attributes": {
                "title": "Первая тестовая вакансия",
                "description": "Описание первой тестовой вакансии",
                "status": "active",
                "lat": 55.7346276065292,
                "long": 37.5734168663621,
                "address": "Москва, улица Еланского, 2",
                "salary_from": null,
                "salary_to": null,
                "salary_period": null,
                "expires_at": "2019-03-11T9:21:19.001+03:00",
                "current_publication_type": "job_publish"                
            }
        },
        {
            "id": "270",
            "type": "job",
            "attributes": {
                "title": "Вторая тестовая вакансия",
                "description": "Описание второй тестовой вакансии",
                "status": "active",
                "lat": 55.7346276065292,
                "long": 37.5734168663621,
                "address": "Москва, улица Еланского, 2",
                "salary_from": 10000,
                "salary_to": 50000,
                "salary_period": "monthly",
                "expires_at": "2019-03-14T10:37:22.813+03:00",
                "remote": false,
                "current_publication_type": "job_publish"
            }
        }
    ]
}
```

### Создание вакансии

Для создания вакансии необходимо отправить POST-запрос на `/api/external/v1/jobs`. В ответе будет либо JSON-представление созданной вакансии, либо сообщение об ошибке.

Пример успешного создания вакансии:

```
$ curl -i -H 'Authorization: Worki <JWT Token>' https://api.iconjob.co/api/external/v1/jobs -H 'Content-Type: application/json' -d '{"job": { "title": "Тестовая вакансия", "description": "Тестовое описание", "lat": 55.35, "long": 37.55, "address": "Москва. Солянка ул., 9", "remote": true}, "profession_id": 3, "publication_type": "super_job_publish"}'

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"data":{"id":"651","type":"job","attributes":{"title": "Тестовая вакансия", "description": "Тестовое описание", "lat": 55.35, "long": 37.55, "address": "Москва. Солянка ул., 9","salary_from":null,"salary_to":null,"salary_period":null,"expires_at": "2019-04-11T12:41:29.237+03:00", "remote": true, "current_publication_type": "super_job_publish"}}}
```

Пример ошибки при создании вакансии:

```
$ curl -i -H 'Authorization: Worki <JWT Token>' https://api.iconjob.co/api/external/v1/jobs -H 'Content-Type: application/json' -d '{"job": { "title": "test title" }}'

HTTP/1.1 422 Unprocessable Entity
Content-Type: application/json; charset=utf-8

{"errors":[{"detail":"Возникли ошибки: Поле 'Адрес' не может быть пустым"}]}

```

### Изменение вакансии

Для изменения вакансии необходимо отправить PUT-запрос на `/api/external/v1/jobs/<id>`, где <id> - идентификатор вакансии. В ответе будет либо JSON-представление созданной вакансии, либо сообщение об ошибке.

Пример успешного обновления вакансии:

```
$ curl -i -H 'Authorization: Worki <JWT Token>' https://api.iconjob.co/api/external/v1/jobs/122 -H 'Content-Type: application/json' -d '{"job": { "title": "Новая тестовая вакансия" }}'

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"data":{"id":"122","type":"job","attributes":{"title": "Новая тестовая вакансия", "description": "Тестовое описание", "lat": 55.35, "long": 37.55, "address": "Москва. Солянка ул., 9","salary_from":null,"salary_to":null,"salary_period":null,"expires_at": "2019-03-14T22:17:56.961+03:00","remote": true}}}
```

### Получение информации о конкретной вакансии

Для получения информации о конкретной вакансии, необходимо отправить GET-запрос на `/api/external/v1/jobs/<id>`, где <id> - идентификатор вакансии. В ответе будет JSON-представление вакансии либо сообщение об ошибке, если такой вакансии не существует.

```$ curl -i -H 'Authorization: Worki <JWT Token>' https://api.iconjob.co/api/external/v1/jobs/254

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "data": [
        {
            "id": "254",
            "type": "job",
            "attributes": {
                "title": "Первая тестовая вакансия",
                "description": "Описание первой тестовой вакансии",
                "status": "active",
                "lat": 55.7346276065292,
                "long": 37.5734168663621,
                "address": "Москва, улица Еланского, 2",
                "salary_from": null,
                "salary_to": null,
                "salary_period": null,
                "expires_at": "2019-03-22T18:02:13.462+03:00",
                "remote": true
            }
        }
}
```

### Закрытие вакансии

Для того чтобы снять вакансию из ленты, необходимо отправить POST-запрос вида `/api/external/v1/jobs/<id>/close`, где `<id>` - идентификатор вакансии, а также необязательный параметр `reason`, который может принимать одно из следующих значений:

* `closed_irrelevant` - вакансия более неактуальна (по-умолчанию)
* `closed_found` - вакансия закрыта, кандидат найден
* `closed_found_other` - вакансия закрыта, кандидат найден в другом месте


В ответе будет JSON-представление закрытой вакансии либо сообщение об ошибке.


### Продление вакансии

Для того чтобы продлить вакансию, необходимо отправить POST-запрос вида `/api/external/v1/jobs/<id>/revive`, где `<id>` - идентификатор вакансии, а также необязательный параметр `publication_type`, который может принимать одно из следующих значений:

* `job_publish` - простая публикация (по-умолчанию, если не указывать тип публикации)
* `super_job_publish` - продвинутая публикация
* `ultra_job_publish` - премиум публикация

В ответе будет JSON-представление продлённой вакансии либо сообщение об ошибке.

## Получение списка категорий профессий

Для получения списка категорий профессий необходимо отправить GET-запрос на `/api/external/v1/categories` (аутентификация не требуется).

Результатом будет список категорий профессий (JSON API-ответ сжат и отформатирован для наглядности):

```
$ curl -i https://api.iconjob.co/api/external/v1/categories

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "data": [
    {
      "id": "1",
      "type": "category",
      "attributes": {
        "id": 1,
        "name": "Общепит",
        "transliterated_name": "obshchepit",
        "popular": true,
        "sorting_value": 24,
        "salary_to": 150000,
        "icon_image_url": "https://hb.bizmrg.com/worki-production/uploads/category/icon_image/1/restaurants.png",
        "alternative_icon_image_url": null
      }
    },
    {
      "id": "2",
      "type": "category",
      "attributes": {
        "id": 10,
        "name": "Магазины",
        "transliterated_name": "magaziny",
        "popular": true,
        "sorting_value": 43,
        "salary_to": null,
        "icon_image_url": "https://hb.bizmrg.com/worki-staging/uploads/category/icon_image/2/retail.png",
        "alternative_icon_image_url": null
      }
    },
    ...
  ]
}

```

## Получение списка профессий

Для получения списка профессий необходимо отправить GET-запрос на `/api/external/v1/professions`. Результатом будет список профессий с идентификаторами, которые необходимо использовать при создании вакансии:

```
$ curl -i -H 'Authorization: Worki <JWT Token>' https://api.iconjob.co/api/external/v1/professions

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "professions": [
        {
            "id": 85,
            "title": "актер"
        },
        {
            "id": 93,
            "title": "массовка"
        },
        {
            "id": 79,
            "title": "рекламщик"
        },
        {
            "id": 87,
            "title": "GO-GO DANCE"
        },
        {
            "id": 74,
            "title": "IT-специалист"
        }
    ]
}

```

## Получение списка регионов

Для получения списка регионов необходимо отправить GET-запрос на `/api/external/v1/regions`. Результатом будет список регионов с идентификаторами:

```
$ curl -i -H 'Authorization: Worki <JWT Token>' https://api.iconjob.co/api/external/v1/regions

HTTP/2 200
Content-Type: application/json; charset=utf-8

{
    "regions": [
        {
            "name": "Вся Россия",
            "fias_id": "00000000-0000-0000-0000-000000000000"
        },
        {
            "name": "Москва",
            "fias_id": "0c5b2444-70a0-4932-980c-b4dc0d3f02b5"
        },
        {
            "name": "Московская область",
            "fias_id": "29251dcf-00a1-4e34-98d4-5c47484a36d4"
        },
        ...
    ]
}

```

## Получение списка городов

Для получения списка городов необходимо отправить GET-запрос на `/api/external/v1/cities`. Результатом будет список городов с идентификаторами (JSON API-ответ сжат и отформатирован для наглядности):

```
$ curl -i -H 'Authorization: Worki <JWT Token>' https://api.iconjob.co/api/external/v1/cities

HTTP/2 200
Content-Type: application/json; charset=utf-8

{
  "data": [
    {
        "id": "286214595",
        "type": "city",
        "attributes": {
            "id": 286214595,
            "region_id": 19,
            "name": "Абаза"
        }
    },
    {
        "id": "191657470",
        "type": "city",
        "attributes": {
            "id": 191657470,
            "region_id": 19,
            "name": "Абакан"
        }
    },
    ...
  ]
}
```

## Получение списка гражданств

Для получения списка поддерживаемых гражданств необходимо отправить GET-запрос на `/api/external/v1/nationalities`. Результатом будет список гражданств с идентификаторами:

```
$ curl -i -H 'Authorization: Worki <JWT Token>' https://api.iconjob.co/api/external/v1/nationalities

HTTP/2 200
Content-Type: application/json; charset=utf-8

{
    "data": [
        {
            "id": "1",
            "type": "nationality",
            "attributes": {
                "id": 1,
                "name": "Российская Федерация"
            }
        },
        {
            "id": "3",
            "type": "nationality",
            "attributes": {
                "id": 3,
                "name": "Азербайджан"
            }
        },
        {
            "id": "5",
            "type": "nationality",
            "attributes": {
                "id": 5,
                "name": "Армения"
            }
        },
        {
            "id": "4",
            "type": "nationality",
            "attributes": {
                "id": 4,
                "name": "Республика Беларусь"
            }
        },
        {
            "id": "6",
            "type": "nationality",
            "attributes": {
                "id": 6,
                "name": "Грузия"
            }
        },
        {
            "id": "7",
            "type": "nationality",
            "attributes": {
                "id": 7,
                "name": "Казахстан"
            }
        },
        {
            "id": "8",
            "type": "nationality",
            "attributes": {
                "id": 8,
                "name": "Киргизия"
            }
        },
        {
            "id": "9",
            "type": "nationality",
            "attributes": {
                "id": 9,
                "name": "Молдавия"
            }
        },
        {
            "id": "10",
            "type": "nationality",
            "attributes": {
                "id": 10,
                "name": "Таджикистан"
            }
        },
        {
            "id": "11",
            "type": "nationality",
            "attributes": {
                "id": 11,
                "name": "Узбекистан"
            }
        },
        {
            "id": "12",
            "type": "nationality",
            "attributes": {
                "id": 12,
                "name": "Украина"
            }
        },
        {
            "id": "13",
            "type": "nationality",
            "attributes": {
                "id": 13,
                "name": "Другое"
            }
        }
    ],
    "links": {
        "first": "http://api.staging.iconjob.co/api/external/v1/nationalities?page%5Bnumber%5D=1",
        "last": "http://api.staging.iconjob.co/api/external/v1/nationalities?page%5Bnumber%5D=1"
    }
}

```

## Отклики

### Получение списка откликов

Несмотря на то, что при создании нового отклика API VK Работа производит запрос во внешнюю учетную систему, для синхронизации разных источников данных можно получить список всех откликов на все вакансии.

Для получения списка откликов необходимо отправить GET-запрос на `/api/external/v1/applications`.

Опциональные параметры:
* `page` - параметры для пагинации:
    * `number` - номер страницы
    * `size` - количество элементов на странице (не более 50)

Результат будет представлен в следующем виде (JSON API-ответ сжат и отформатирован для наглядности):

```
$ curl -i -H 'Authorization: Worki <JWT Token>' https://api.iconjob.co/api/external/v1/applications

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "data": [
        {
            "id": "1",
            "type": "application",
            "attributes": {
                "id": 1,
                "recruiter_id": 888,
                "job_id": 298,
                "status": "reserved",
                "created_at": "2017-10-11T20:57:43.411+03:00"
            },
            "relationships": {
                "candidate": {
                    "data": {
                        "id": "889",
                        "type": "candidate"
                    }
                }
            }
        },
        {
            "id": "2",
            "type": "application",
            "attributes": {
                "id": 2,
                "recruiter_id": 888,
                "job_id": 648,
                "status": "read",
                "created_at": "2018-06-21T21:10:43.256+03:00"
            },
            "relationships": {
                "candidate": {
                    "data": {
                        "id": "889",
                        "type": "candidate"
                    }
                }
            }
        },
    ],
    "included": [
        {
            "id": "889",
            "type": "candidate",
            "attributes": {
                "id": 889,
                "first_name": "Иван",
                "last_name": "Петров",
                "phone": "79995225324",
                "description": "Небольшой текст о себе",
                "lat": 55.7346276065292,
                "long": 37.5734168663621,
                "address": "Москва, улица Еланского, 2",
                "education": "Среднее образование",
                "languages": "Pусский",
                "last_online_time": "2017-06-28 11:17:13 +0300",
                "created_at": "2017-09-30T17:03:46.313+03:00",
                "updated_at": "2017-10-10T20:00:54.754+03:00",
                "email": null,
                "birthday": null,
                "driving_license_categories": [],
                "medical_record": null,
                "active_candidate": true,
                "accept_calls": false
            },
            "relationships": {
                "avatar": {
                    "data": null
                },
                "background": {
                    "data": null
                },
                "experiences": {
                    "data": [
                        {
                            "id": "360",
                            "type": "experience"
                        },
                        {
                            "id": "361",
                            "type": "experience"
                        }
                    ]
                },
                "nationality": {
                    "data": null
                }
            }
        },
        {
            "id": "360",
            "type": "experience",
            "attributes": {
                "title": "Первое место работы",
                "description": "Чем-то там занимался",
                "duration": null,
                "current": true
            },
            "relationships": {
                "profession": {
                    "data": null
                }
            }
        },
        {
            "id": "361",
            "type": "experience",
            "attributes": {
                "title": "Второе место работы",
                "description": "Чем-то и на нем занимался",
                "duration": null,
                "current": false
            },
            "relationships": {
                "profession": {
                    "data": null
                }
            }
        }
    ]
}
```

Также можно воспользоваться фильтром по вакансии, передав параметр `job_id`:

```
$ curl -i -H 'Authorization: Worki <JWT Token>' https://api.iconjob.co/api/external/v1/applications?job_id=199

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "data": [
        {
            "id": "1",
            "type": "application",
            "attributes": {
                "id": 1,
                "recruiter_id": 888,
                "job_id": 199,
                "status": "reserved",
                "created_at": "2017-10-11T20:57:43.411+03:00"
            },
            "relationships": {
                "candidate": {
                    "data": {
                        "id": "889",
                        "type": "candidate"
                    }
                }
            }
        },
    ],
    "included": [
        {
            "id": "889",
            "type": "candidate",
            "attributes": {
                "id": 889,
                "first_name": "Иван",
                "last_name": "Петров",
                "phone": "79995225324",
                "description": "Небольшой текст о себе",
                "lat": 55.7346276065292,
                "long": 37.5734168663621,
                "address": "Москва, улица Еланского, 2",
                "education": "Среднее образование",
                "languages": "Pусский",
                "last_online_time": "2017-06-28 11:17:13 +0300",
                "created_at": "2017-09-30T17:03:46.313+03:00",
                "updated_at": "2017-10-10T20:00:54.754+03:00",
                "email": null,
                "birthday": null,
                "driving_license_categories": [],
                "medical_record": null,
                "active_candidate": true,
                "accept_calls": false
            },
            "relationships": {
                "avatar": {
                    "data": null
                },
                "background": {
                    "data": null
                },
                "experiences": {
                    "data": [
                        {
                            "id": "360",
                            "type": "experience"
                        },
                        {
                            "id": "361",
                            "type": "experience"
                        }
                    ]
                },
                "nationality": {
                    "data": null
                }
            }
        },
        {
            "id": "360",
            "type": "experience",
            "attributes": {
                "title": "Первое место работы",
                "description": "Чем-то там занимался",
                "duration": null,
                "current": true
            },
            "relationships": {
                "profession": {
                    "data": null
                }
            }
        },
        {
            "id": "361",
            "type": "experience",
            "attributes": {
                "title": "Второе место работы",
                "description": "Чем-то и на нем занимался",
                "duration": null,
                "current": false
            },
            "relationships": {
                "profession": {
                    "data": null
                }
            }
        }
    ]
}
```

Также можно воспользоваться фильтром по статусу отклика, передав параметр `status` (допускается передавать массив статусов):

```
$ curl -i -H 'Authorization: Worki <JWT Token>' https://api.iconjob.co/api/external/v1/applications?status[]=reserved&status[]=improper

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "data": [
        {
            "id": "1",
            "type": "application",
            "attributes": {
                "id": 1,
                "recruiter_id": 888,
                "job_id": 298,
                "status": "reserved",
                "created_at": "2017-10-11T20:57:43.411+03:00"
            },
            "relationships": {
                "candidate": {
                    "data": {
                        "id": "889",
                        "type": "candidate"
                    }
                }
            }
        },
    ],
    "included": [
        {
            "id": "889",
            "type": "candidate",
            "attributes": {
                "id": 889,
                "first_name": "Иван",
                "last_name": "Петров",
                "phone": "79995225324",
                "description": "Небольшой текст о себе",
                "lat": 55.7346276065292,
                "long": 37.5734168663621,
                "address": "Москва, улица Еланского, 2",
                "education": "Среднее образование",
                "languages": "Pусский",
                "last_online_time": "2017-06-28 11:17:13 +0300",
                "created_at": "2017-09-30T17:03:46.313+03:00",
                "updated_at": "2017-10-10T20:00:54.754+03:00",
                "email": null,
                "birthday": null,
                "driving_license_categories": [],
                "medical_record": null,
                "active_candidate": true,
                "accept_calls": false
            },
            "relationships": {
                "avatar": {
                    "data": null
                },
                "background": {
                    "data": null
                },
                "experiences": {
                    "data": [
                        {
                            "id": "360",
                            "type": "experience"
                        },
                        {
                            "id": "361",
                            "type": "experience"
                        }
                    ]
                },
                "nationality": {
                    "data": null
                }
            }
        },
        {
            "id": "360",
            "type": "experience",
            "attributes": {
                "title": "Первое место работы",
                "description": "Чем-то там занимался",
                "duration": null,
                "current": true
            },
            "relationships": {
                "profession": {
                    "data": null
                }
            }
        },
        {
            "id": "361",
            "type": "experience",
            "attributes": {
                "title": "Второе место работы",
                "description": "Чем-то и на нем занимался",
                "duration": null,
                "current": false
            },
            "relationships": {
                "profession": {
                    "data": null
                }
            }
        }
    ]
}
```

Или же фильтровать отклики по дате создания, используя параметр `created_from`:

```
$ curl -i -H 'Authorization: Worki <JWT Token>' https://api.iconjob.co/api/external/v1/applications?created_from=2019-03-01+00%3A00%3A00

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "data": [
        {
            "id": "12",
            "type": "application",
            "attributes": {
                "id": 12,
                "recruiter_id": 888,
                "job_id": 298,
                "status": "reserved",
                "created_at": "2019-03-07T20:57:43.411+03:00"
            },
            "relationships": {
                "candidate": {
                    "data": {
                        "id": "889",
                        "type": "candidate"
                    }
                }
            }
        },
    ],
    "included": [
        {
            "id": "889",
            "type": "candidate",
            "attributes": {
                "id": 889,
                "first_name": "Иван",
                "last_name": "Петров",
                "phone": "79995225324",
                "description": "Небольшой текст о себе",
                "lat": 55.7346276065292,
                "long": 37.5734168663621,
                "address": "Москва, улица Еланского, 2",
                "education": "Среднее образование",
                "languages": "Pусский",
                "last_online_time": "2017-06-28 11:17:13 +0300",
                "created_at": "2017-09-30T17:03:46.313+03:00",
                "updated_at": "2017-10-10T20:00:54.754+03:00",
                "email": null,
                "birthday": null,
                "driving_license_categories": [],
                "medical_record": null,
                "active_candidate": true,
                "accept_calls": false
            },
            "relationships": {
                "avatar": {
                    "data": null
                },
                "background": {
                    "data": null
                },
                "experiences": {
                    "data": [
                        {
                            "id": "360",
                            "type": "experience"
                        },
                        {
                            "id": "361",
                            "type": "experience"
                        }
                    ]
                },
                "nationality": {
                    "data": null
                }
            }
        },
        {
            "id": "360",
            "type": "experience",
            "attributes": {
                "title": "Первое место работы",
                "description": "Чем-то там занимался",
                "duration": null,
                "current": true
            },
            "relationships": {
                "profession": {
                    "data": null
                }
            }
        },
        {
            "id": "361",
            "type": "experience",
            "attributes": {
                "title": "Второе место работы",
                "description": "Чем-то и на нем занимался",
                "duration": null,
                "current": false
            },
            "relationships": {
                "profession": {
                    "data": null
                }
            }
        }
    ]
}
```

### Изменение статуса отклика

Для изменения статуса отклика необходимо сделать POST-запрос на `/api/external/v1/applications/<application_id>/proceed`. В ответе будет JSON API представление выбранного отклика либо сообщение об ошибке. В параметрах необходимо передать статус отклика, который может принимать одно из значений атрибута `status` модели `Application`, описанных выше.

Пример запроса с успешным изменением статуса:

```$ curl -i -XPOST https://api.iconjob.co/api/external/v1/applications/1/proceed -d '{"application":{"status": "invited"}}' -H 'Authorization: Worki <JWT Token>' -H "Content-Type: application/json"

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "data": {
        "id": "1",
        "type": "application",
        "attributes": {
            "id": 1,
            "recruiter_id": 888,
            "job_id": 298,
            "status": "invited",
            "created_at": "2017-10-11T20:57:43.411+03:00"
        },
        "relationships": {
            "candidate": {
                "data": {
                    "id": "889",
                    "type": "candidate"
                }
            }
        }
    },
    "included": [
        {
            "id": "889",
            "type": "candidate",
            "attributes": {
                "id": 889,
                "first_name": "Иван",
                "last_name": "Петров",
                "phone": "79995225324",
                "description": "Небольшой текст о себе",
                "lat": 55.7346276065292,
                "long": 37.5734168663621,
                "address": "Москва, улица Еланского, 2",
                "education": "Среднее образование",
                "languages": "Pусский",
                "last_online_time": "2017-06-28 11:17:13 +0300",
                "created_at": "2017-09-30T17:03:46.313+03:00",
                "updated_at": "2017-10-10T20:00:54.754+03:00",
                "email": null,
                "birthday": null,
                "driving_license_categories": [],
                "medical_record": null,
                "active_candidate": true,
                "accept_calls": false
            },
            "relationships": {
                "avatar": {
                    "data": null
                },
                "background": {
                    "data": null
                },
                "experiences": {
                    "data": [
                        {
                            "id": "360",
                            "type": "experience"
                        },
                        {
                            "id": "361",
                            "type": "experience"
                        }
                    ]
                },
                "nationality": {
                    "data": null
                }
            }
        },
        {
            "id": "360",
            "type": "experience",
            "attributes": {
                "title": "Первое место работы",
                "description": "Чем-то там занимался",
                "duration": null,
                "current": true
            },
            "relationships": {
                "profession": {
                    "data": null
                }
            }
        },
        {
            "id": "361",
            "type": "experience",
            "attributes": {
                "title": "Второе место работы",
                "description": "Чем-то и на нем занимался",
                "duration": null,
                "current": false
            },
            "relationships": {
                "profession": {
                    "data": null
                }
            }
        }
    ]
}
```

Пример запроса с неуспешным изменением статуса:

```$ curl -i -XPOST https://api.iconjob.co/api/external/v1/applications/1/proceed -H 'Authorization: Worki <JWT Token>' -H 'Content-Type: application/json' -d '{"application": { "status": "not_exists" }}'

HTTP/1.1 422 Unprocessable Entity
Content-Type: application/json; charset=utf-8

{"errors":[{"detail":"Неизвестный статус"}]}
```

## Баланс компании и VAS'ы

### Получение состояния баланса компании

Для получения баланса VAS'ов необходимо отправить GET-запрос на `/api/external/v1/company`.

Результатом будет в следующем виде (JSON API-ответ отформатирован для наглядности)

```
curl -i -H 'Authorization: Worki <JWT Token>' https://api.iconjob.co/api/external/v1/company

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
{
    "data": {
        "id": "1",
        "type": "company",
        "attributes": {
            "id": 1,
            "otvas_expires_at": null,
            "balance": "10000000.0",
            "wallet_id": null,
            "otvas_user_limit": 1,
            "otvas_jobs_limit": 0,
            "otvas_refreshes_limit": 1,
            "otvas_highlights_limit": 0,
            "otvas_views_limit": 1,
            "otvas_elevations_plus_limit": 1,
            "otvas_elevations_1d_limit": 0,
            "otvas_elevations_3d_limit": 0
        }
    }
}

```

### Получение списка транзакций VAS'ов

Для получения списка транзакций необходимо отправить GET-запрос на `/api/external/v1/vas_transactions`.

Опциональные параметры:
* `page` - параметры для пагинации:
    * `number` - номер страницы
    * `size` - количество элементов на странице (не более 50)

Результатом будет в следующем виде (JSON API-ответ отформатирован для наглядности)

```
$ curl -i -H 'Authorization: Worki <JWT Token>' https://api.iconjob.co/api/external/v1/vas_transactions?page[number]=1&page[size]=10

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
{
    "data": [
        {
            "id": "4",
            "type": "vas_transaction",
            "attributes": {
                "created_at": "2011-11-11T00:00:00.000+04:00",
                "updated_at": "2011-11-11T00:00:00.000+04:00",
                "subscriptable_type": "OTVAS",
                "creator": "Внешний пользователь(ibs) #4",
                "applied_changes": [
                    "Размещение вакансии: -1"
                ]
            }
        },
        {
            "id": "5",
            "type": "vas_transaction",
            "attributes": {
                "created_at": "2011-11-11T00:00:00.000+04:00",
                "updated_at": "2011-11-11T00:00:00.000+04:00",
                "subscriptable_type": "OTVAS",
                "creator": "Внешний пользователь(ibs) #4",
                "applied_changes": [
                    "Выделение вакансии: -1"
                ]
            }
        }
    ],
    "meta": {
        "total": 2
    }
}
```

### Применение VAS'ов

Для применения VAS'ов необходимо отправить POST-запрос на `/api/external/v1/paid_actions`.
Параметры:
- `actions` - применяемые действия, массив
    - `action` - действие, которое применяется. Желательно брать код из раздела "Аббревиатуры".
    - `job_id` - id вакансии, к которому применяется VAS (если необходимо)
    - `user_id` - id кандидата, с кем создается диалог/чей телефон просматривается (если необходимо)
- `use_account` - баланс, с которого идет списание. Принимает значения `bonus` (оплата бонусами), `company` (оплата с кошелька). По умолчанию `company`

В ответе возвращаются применённые действия и текущее состояние подписки(поле `meta`->`company`).

Результатом будет в следующем виде (JSON API-ответ отформатирован для наглядности)

```
$ curl -i -H 'Authorization: Worki <JWT Token>' -d '{"actions":[{"action":"job_highlight", "job_id": "1"}]}' -H "Content-Type: application/json" -X POST https://api.iconjob.co/api/external/v1/paid_actions

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
{
    "data": [
        {
            "id": "0",
            "type": "paid_action",
            "attributes": {
                "id": 0,
                "error_message": null,
                "status": "success",
                "origin": {
                    "action": "job_highlight",
                    "count": 1,
                    "job_id": "1"
                },
                "job": {
                    "data": {
                        "id": "1",
                        "type": "job",
                        "attributes": {
                            "title": "job_title_1",
                            "description": "Приходите работать в Бабушкины яички",
                            "status": "active",
                            "lat": 11.23,
                            "long": 31.53,
                            "address": "Москва, туту",
                            "expires_at": "2011-11-12T00:00:00.000+04:00",
                            "salary_from": null,
                            "salary_to": null,
                            "salary_period": null,
                            "contact_phone": null,
                            "call_only": false,
                            "auto_renew": null,
                            "refreshed_at": "2011-11-11T00:00:00.000+04:00",
                            "refreshable": false,
                            "refreshable_at": "2011-11-16T00:00:00.000+04:00",
                            "today_views_count": 0,
                            "external_id": null,
                            "revivable": true,
                            "published_at": "2011-11-11T00:00:00.000+04:00",
                            "highlighted": true,
                            "highlighted_till": "2011-11-12T00:00:00.000+04:00",
                            "parttime": false,
                            "watch": false,
                            "no_experience": false,
                            "disabilities": false,
                            "public_link": "https://worki.ru/vacancy/WJAUJZ"
                        },
                        "relationships": {
                            "profession": {
                                "data": {
                                    "id": "1",
                                    "type": "profession"
                                }
                            }
                        }
                    }
                }
            }
        }
    ],
    "meta": {
        "company": {
            "data": {
                "id": "1",
                "type": "company",
                "attributes": {
                    "id": 1,
                    "otvas_expires_at": null,
                    "balance": "10000000.0",
                    "wallet_id": null,
                    "otvas_user_limit": 1,
                    "otvas_jobs_limit": 0,
                    "otvas_refreshes_limit": 1,
                    "otvas_highlights_limit": 0,
                    "otvas_views_limit": 1,
                    "otvas_elevations_plus_limit": 1,
                    "otvas_elevations_1d_limit": 0,
                    "otvas_elevations_3d_limit": 0
                }
            }
        }
    }
}
```

## Поиск и покупка контактов из базы резюме

### Поиск кандидатов

Для поиска кандидатов необходимо отправить GET-запрос на `/api/external/v1/candidates`.

Обязательно наличие одного из вариантов фильтров:
* `address` - адрес (будет декодирован в координаты. Без указания параметра `distance_to` будет произведен поиск в радиусе 15 км)
* `city_id` - id города из [справочника городов](#получение-списка-городов)
* `region_id` - id региона из [справочника регионов](#получение-списка-регионов)
* `lat`, `long`, `distance_to` - координаты и радиус поиска (в км)

Список опциональных фильтров:
* `page` - страница пагинации (по-умолчанию `1`)
* `per_page` - количество элементов на странице (от `1` до `50`, по-умолчанию `50`)
* `distance_to` - радиус поиска (в км)
* `month_of_experience` - опыт работы (в месяцах)
* `key_word` - ключевые слова
* `nationality_id` - гражданство (id страны, см. [справочник гражданств](#получение-списка-гражданств))
* `has_avatar` - только с фото (1 - есть, 0 - без указания)
* `medical_record` - есть мед. книжка (1 - есть, 0 - без указания)
* `sex` - пол (Male - мужской, Female - женский)
* `has_comment` - резюме с заметками (1 - есть, 0 - без указания)
* `address` - адрес, рядом с которым ведется поиск
* `show_bought` - только купленные контакты (1 - есть, 0 - без указания)
* `driving_license_categories` - категории водительских прав (доступные значения A, A1, B, BE, B1, C, CE, C1, C1E, D, DE, D1, D1E, M, TM, TB)
* `age_from` - возраст от (допустимый диапазон 14 - 99)
* `age_to` - возраст до (допустимый диапазон 14 - 99)
* `key_word` - ключевые слова

Результат будет в следующем виде (JSON API-ответ сжат и отформатирован для наглядности):

```
$ curl -i -H 'Authorization: Worki <JWT Token>' https://api.iconjob.co/api/external/v1/candidates?address=Москва

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "data": [
    {
      "id": "29638",
      "type": "candidate_search",
      "attributes": {
        "id": 29638,
        "first_name": "Иван",
        "last_name": "Иванов",
        "description": "Небольшой текст о себе",
        "lat": 56.3209074862824,
        "long": 44.0091540738274,
        "education": "Среднее образование",
        "languages": "Pусский",
        "online": false,
        "last_online_at": "2021-09-14T01:48:42.391+03:00",
        "created_at": "2020-07-22T16:21:08.495+03:00",
        "updated_at": "2021-09-14T01:53:13.227+03:00",
        "views_count": 43,
        "driving_license_categories": [
          "A",
          "A1",
          "B",
          "BE",
          "B1",
          "C",
          "CE",
          "C1",
          "C1E",
          "D",
          "DE",
          "D1",
          "D1E",
          "M",
          "TM",
          "TB"
        ],
        "medical_record": false,
        "candidate_status": "candidate_active",
        "active_candidate": true,
        "comment": {
          "data": null
        },
        "comment_count": 0,
        "has_vk_client": false,
        "already_contacted": true,
        "highlighted_till": null,
        "avatar": {
          "data": {
            "id": "5312",
            "type": "candidate_avatar",
            "attributes": {
              "id": 5312,
              "content_type": "image/jpeg",
              "small": "https://hb.bizmrg.com/worki-staging/static/closed_contact_96x96.png",
              "medium": "https://hb.bizmrg.com/worki-staging/static/closed_contact_165x165.png",
              "original": "https://hb.bizmrg.com/worki-staging/static/closed_contact_165x165.png"
            }
          }
        },
        "background": null,
        "email_sendable?": false,
        "banned": false,
        "deleted?": false,
        "disability_group": 0,
        "vaccinated": false,
        "city_name": "Нижний Новгород",
        "certificates": {
          "skillbox": true
        },
        "age": 19,
        "viewing_contacted_recruiter": true,
        "view_by_himself": false
      },
      "relationships": {
        "professions": {
          "data": []
        },
        "preferred_professions": {
          "data": []
        },
        "nationality": {
          "data": {
            "id": "1",
            "type": "nationality"
          }
        },
        "experiences": {
          "data": []
        }
      }
    },
    ...
  ],
  "included": [
    {
      "id": "1",
      "type": "nationality",
      "attributes": {
        "id": 1,
        "name": "Российская Федерация"
      }
    },
    {
      "id": "94",
      "type": "profession",
      "attributes": {
        "id": 94,
        "title": "Курьер"
      }
    },
    ...
  ],
  "meta": {
    "total": 3443,
    "total_all_time": 3443
  }
}
```

### Покупка контакта

Для покупки контакта кандидата воспользуйтесь методом [`/api/external/v1/paid_actions`](#применение-vasов).

Параметры:
- `actions` - применяемые действия, массив
    - `action` - действие, передайте значение `contact_view`
    - `user_id` - id кандидата, чей телефон просматривается
- `use_account` - баланс, с которого идет списание. Принимает значения `bonus` (оплата бонусами), `company` (оплата с кошелька). По умолчанию `company`

```
$ curl -i -H 'Authorization: Worki <JWT Token>' -d '{"actions":[{"action":"contact_view", "user_id": "5657"}]}' -H "Content-Type: application/json" -X POST https://api.iconjob.co/api/external/v1/paid_actions

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "data":[
    {
      "id":"0",
      "type":"paid_action",
      "attributes":{
        "id":0,
        "error_message":null,
        "status":"success",
        "origin":{
          "action":"contact_view",
          "count":1,
          "job_id":null,
          "user_id":"5657",
          "price":1500,
          "region_id":80
        },
        "job":null,
        "user":{
          "data":{
            "id":"5657",
            "type":"candidate",
            "attributes":{
              "id":5657,
              "first_name":"имя",
              "last_name":"фамилия",
              "phone":"75345345345",
              "description":null,
              "lat":59.9845226,
              "long":30.2447583,
              "years_of_experience":null,
              "education":null,
              "languages":null,
              "last_online_time":"2021-06-28T15:15:37.776+03:00",
              "created_at":"2019-09-23T17:21:45.211+03:00",
              "updated_at":"2019-09-23T17:25:30.025+03:00",
              "email":null,
              "birthday":"1999-11-11",
              "driving_license_categories":[
                
              ],
              "medical_record":null,
              "active_candidate":true,
              "accept_calls":false,
              "address":null,
              "city_name":"Санкт-Петербург"
            },
            "relationships":{
              "avatar":{
                "data":null
              },
              "background":{
                "data":null
              },
              "experiences":{
                "data":[
                    
                ]
              },
              "nationality":{
                "data":{
                  "id":"1",
                  "type":"nationality"
                }
              }
            }
          }
        }
      }
    }
  ],
  "meta":{
    "company":{
      "data":{
        "id":"17282",
        "type":"company",
        "attributes":{
          "id":17282,
          "balance":"67500.0",
          "wallet_id":"W843633",
          "otvas_jobs_limit":0,
          "otvas_super_jobs_limit":0,
          "otvas_ultra_jobs_limit":0,
          "otvas_refreshes_limit":0,
          "otvas_highlights_limit":0,
          "otvas_views_limit":0,
          "otvas_elevations_plus_limit":0,
          "otvas_elevations_1d_limit":0,
          "otvas_elevations_3d_limit":0,
          "otvas_expires_at":null
        }
      }
    }
  }
}
```

### Просмотр контакта

Для просмотра купленного контакта необходимо отправить GET-запрос на `/api/external/v1/candidates/<id>/phone`, где <id> - идентификатор кандидата.

**Важно!** В случае если контакт не куплен, произойдет автоматическое списание средств!

Результат будет в следующем виде (JSON API-ответ отформатирован для наглядности):

```
$ curl -i -H 'Authorization: Worki <JWT Token>' https://api.iconjob.co/api/external/v1/candidates/31136/phone

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "data": {
    "id": "31136",
    "type": "user",
    "attributes": {
      "phone": "79001111111",
      "contact_bought": true,
      "already_contacted": false,
      "spent_counts": {
        "count_for_packets": 1,
        "count_for_subscription": 0,
        "count_for_company": 0,
        "count_for_bonus": 0,
        "count_for_services": 0
      },
      "packet_used": true,
      "contact_used": true,
      "subscription": null
    }
  }
}
```

## Фоновое изображение

### Загрузка фонового изображения

Для добавления фонового изображения к необходимо отправить PUT-запрос на `/api/external/v1/background` с  заголовком `Content-Type: multipart/form-data`.

Обязательные параметры:
    * data - файл с изображением
Опциональные параметры:
    * job_ids[] - массив идентификаторов вакансий, при его отсутствии заменится фоновое изображение привязанное к пользователю, под которым размещаются вакансии(при отсутствии у вакансии своего фона, будет использован фон пользователя)
    * crop_h, crop_w - высота и ширина до которых нужно обрезать изображение
    * crop_x, crop_y - что это точка(отчет от левого нижнего угла) с которой будет обрезаться изображение(отсечётся часть где x или y меньше указанных)

Результат будет в следующем виде (JSON API-ответ отформатирован для наглядности)

Обратите внимание, в ответе будет всегда приходить 200, даже если какое-то из изображений не удалось сохранить, о наличии ошибки можно судить по отсутствующему id фонового изображения и наличию непустого поля errors. Если вакансия не принадлежит вашему пользователю или отсутствует, то изображение с соответствующим assetable_id не будет обработано и добавлено в ответ

```
curl --request PUT \
  --url 'https://api.iconjob.co/api/external/v1/background' \
  --header 'authorization: Worki <JWT Token>' \
  --header 'content-type: multipart/form-data; boundary=---011000010111000001101001' \
  --form data=<binary-data-here>\
  --form 'job_ids[]=39085' \
  --form 'job_ids[]=39086' \
  --form crop_h=480 \
  --form crop_w=640 \
  --form crop_x=120 \
  --form crop_y=200

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
{
  "data": [
    {
      "id": "4913",
      "type": "background_extended",
      "attributes": {
        "id": 4913,
        "assetable_type": "Job",
        "assetable_id": 39085,
        "content_type": "image/jpeg",
        "original": "https://example.com/123/bg.jpg",
        "cropped": "https://example.com/124/croped/bg.jpg",
        "errors": {}
      }
    },
    {
      "id": null,
      "type": "background_extended",
      "attributes": {
        "id": null,
        "assetable_type": "Job",
        "assetable_id": 39086,
        "content_type": "image/jpeg",
        "original": "https://example.com/124/bg.jpg",
        "cropped": "https://example.com/124/croped/bg.jpg",
        "errors": {
          "base": [
            "Размер обрезанного изображения меньше допустимого (640×180)"
          ]
        }
      }
    }
  ]
}
```

### Удаление фонового изображения

Для удаления фонового изображения к необходимо отправить DELETE-запрос на `/api/external/v1/background`

Опциональные параметры:
    * job_ids[] - массив идентификаторов вакансий, при его отсутствии удалится фоновое изображение привязанное к пользователю, под которым размещаются вакансии(при отсутствии у вакансии своего фона, будет использован фон пользователя)

Результат будет в следующем виде (JSON API-ответ отформатирован для наглядности)

Обратите внимание, в ответе будет всегда приходить 200. Если вакансия не принадлежит вашему пользователю, отсутствует или не содержит фонового изображение, то изображение с соответствующим assetable_id не будет обработано и добавлено в ответ

```
curl --request DELETE \
  --url 'https://api.iconjob.co/api/external/v1/background' \
  --header 'authorization: Worki <JWT Token>' \
  --header 'content-type: application/json' \
  --cookie __profilin=p%253Dt \
  --data '{ "job_ids": [
        "39085", "39086"
    ]
}'

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
{
  "data": [
    {
      "id": "4904",
      "type": "background_extended",
      "attributes": {
        "id": 4904,
        "assetable_type": "User",
        "assetable_id": 2138,
        "content_type": "image/jpeg",
        "original": "https://example.com/123/bg.jpg",
        "cropped": "https://example.com/123/croped/bg.jpg",
        "errors": {}
      }
    },
    {
      "id": "4905",
      "type": "background_extended",
      "attributes": {
        "id": 4905,
        "assetable_type": "User",
        "assetable_id": 2138,
        "content_type": "image/jpeg",
        "original": "https://example.com/124/bg.jpg",
        "cropped": "https://example.com/124/croped/bg.jpg",
        "errors": {}
      }
    }
  ]
}
```

## Получение списка пакетов

Для получения списка пакетов к необходимо отправить GET-запрос на `/api/external/v1/packets`

Опциональные параметры:
* fias_id — guid региона согласно [ФИАС](https://fias.nalog.ru/)
* status - строка, статус пакета. Должен быть одним из следующих значений: `active`, `archived`

Результат будет в следующем виде (JSON API-ответ отформатирован для наглядности)

```
$ curl -i -H 'Authorization: Worki <JWT Token>' https://api.iconjob.co/api/external/v1/packets

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"packets"=>
  [{"start_time"=>"2021-06-25T11:49:38.515+03:00",
    "end_time"=>"2021-06-26T11:49:38.515+03:00",
    "current_step"=>
     {"start_time"=>"2021-06-25T11:49:38.515+03:00",
      "end_time"=>"2021-06-26T11:49:38.515+03:00",
      "active"=>true,
      "packet_type"=>"job_packet",
      "prolongable"=>false,
      "russian_region"=>"Region 16",
      "hidden"=>false,
      "price"=>nil,
      "fias_id"=>"01d3b0b4-7fca-486a-8b54-73e8b729fd4d",
      "prices"=>
       [{"code"=>"job_publish", "price"=>0},
        {"code"=>"super_job_publish", "price"=>0},
        {"code"=>"ultra_job_publish", "price"=>0},
        {"code"=>"job_refresh", "price"=>0},
        {"code"=>"job_highlight", "price"=>0},
        {"code"=>"contact_view", "price"=>0},
        {"code"=>"job_elevate_plus", "price"=>0},
        {"code"=>"job_elevate_1d", "price"=>0},
        {"code"=>"job_elevate_3d", "price"=>0}],
      "limits"=>
       [{"code"=>"job_publish", "initial"=>0, "left"=>10, "default"=>0},
        {"code"=>"super_job_publish", "initial"=>0, "left"=>0, "default"=>0},
        {"code"=>"ultra_job_publish", "initial"=>0, "left"=>0, "default"=>0},
        {"code"=>"job_refresh", "initial"=>0, "left"=>0, "default"=>0},
        {"code"=>"job_highlight", "initial"=>0, "left"=>0, "default"=>0},
        {"code"=>"contact_view", "initial"=>0, "left"=>0, "default"=>0},
        {"code"=>"job_elevate_plus", "initial"=>0, "left"=>0, "default"=>0},
        {"code"=>"job_elevate_1d", "initial"=>0, "left"=>0, "default"=>0},
        {"code"=>"job_elevate_3d", "initial"=>0, "left"=>0, "default"=>0}],
      "created_at"=>"2021-06-25T11:49:38.879+03:00",
      "initial_views_limit"=>0,
      "views_limit"=>0,
      "buy_transaction_type"=>"BonusTransaction",
      "job_category_uuid"=>"00000000-0000-0000-0000-000000000000"},
    "last_step"=>
     {"start_time"=>"2021-06-25T11:49:38.515+03:00",
      "end_time"=>"2021-06-26T11:49:38.515+03:00",
      "active"=>true,
      "packet_type"=>"job_packet",
      "prolongable"=>false,
      "russian_region"=>"Region 16",
      "hidden"=>false,
      "price"=>nil,
      "fias_id"=>"01d3b0b4-7fca-486a-8b54-73e8b729fd4d",
      "prices"=>
       [{"code"=>"job_publish", "price"=>0},
        {"code"=>"super_job_publish", "price"=>0},
        {"code"=>"ultra_job_publish", "price"=>0},
        {"code"=>"job_refresh", "price"=>0},
        {"code"=>"job_highlight", "price"=>0},
        {"code"=>"contact_view", "price"=>0},
        {"code"=>"job_elevate_plus", "price"=>0},
        {"code"=>"job_elevate_1d", "price"=>0},
        {"code"=>"job_elevate_3d", "price"=>0}],
      "limits"=>
       [{"code"=>"job_publish", "initial"=>0, "left"=>10, "default"=>0},
        {"code"=>"super_job_publish", "initial"=>0, "left"=>0, "default"=>0},
        {"code"=>"ultra_job_publish", "initial"=>0, "left"=>0, "default"=>0},
        {"code"=>"job_refresh", "initial"=>0, "left"=>0, "default"=>0},
        {"code"=>"job_highlight", "initial"=>0, "left"=>0, "default"=>0},
        {"code"=>"contact_view", "initial"=>0, "left"=>0, "default"=>0},
        {"code"=>"job_elevate_plus", "initial"=>0, "left"=>0, "default"=>0},
        {"code"=>"job_elevate_1d", "initial"=>0, "left"=>0, "default"=>0},
        {"code"=>"job_elevate_3d", "initial"=>0, "left"=>0, "default"=>0}],
      "created_at"=>"2021-06-25T11:49:38.879+03:00",
      "initial_views_limit"=>0,
      "views_limit"=>0,
      "buy_transaction_type"=>"BonusTransaction",
      "job_category_uuid"=>"00000000-0000-0000-0000-000000000000"},
    "days_count"=>1,
    "left_days_count"=>1,
    "id"=>151,
    "free"=>nil,
    "auto_prolongable"=>false}],
 "meta"=>
  {"total"=>1,
   "active"=>1,
   "archived"=>0,
   "active_total"=>3,
   "archived_total"=>1,
   "free_total"=>1,
   "paid_total"=>2}}
```


## Получение баланса пакетов

Для получения баланса пакетов необходимо отправить GET-запрос на `/api/external/v1/packets/categories_balance`

Результат будет в следующем виде (JSON API-ответ отформатирован для наглядности)

```
$ curl -i -H 'Authorization: Worki <JWT Token>' https://api.iconjob.co/api/external/v1/packets/categories_balance

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"items"=>
  [{"fias_id"=>"00000000-0000-0000-0000-000000000000",
    "job_category_uuid"=>"00000000-0000-0000-0000-000000000000",
    "balance"=>
     [{"bought"=>10, "free"=>0, "code"=>"job_publish"},
      {"bought"=>0, "free"=>0, "code"=>"super_job_publish"},
      {"bought"=>0, "free"=>0, "code"=>"ultra_job_publish"},
      {"bought"=>0, "free"=>0, "code"=>"job_refresh"},
      {"bought"=>0, "free"=>0, "code"=>"job_highlight"},
      {"bought"=>0, "free"=>0, "code"=>"contact_view"},
      {"bought"=>0, "free"=>0, "code"=>"job_elevate_plus"},
      {"bought"=>0, "free"=>0, "code"=>"job_elevate_1d"},
      {"bought"=>0, "free"=>0, "code"=>"job_elevate_3d"}]},
   {"fias_id"=>"00000000-0000-0000-0000-000000000000",
    "job_category_uuid"=>"38dca0fe-8ecb-4fbb-ac92-85a698b1ab04",
    "balance"=>
     [{"code"=>"job_publish", "free"=>1, "bought"=>0},
      {"code"=>"super_job_publish", "free"=>0, "bought"=>0},
      {"code"=>"ultra_job_publish", "free"=>0, "bought"=>0},
      {"code"=>"job_refresh", "free"=>0, "bought"=>0},
      {"code"=>"job_highlight", "free"=>0, "bought"=>0},
      {"code"=>"contact_view", "free"=>0, "bought"=>0},
      {"code"=>"job_elevate_plus", "free"=>0, "bought"=>0},
      {"code"=>"job_elevate_1d", "free"=>0, "bought"=>0},
      {"code"=>"job_elevate_3d", "free"=>0, "bought"=>0}]}],
 "last_view_access_group_packet"=>
  {"start_time"=>"2021-06-24T13:31:20.134+03:00",
   "end_time"=>"2021-06-26T13:31:20.134+03:00",
   "current_step"=>
    {"start_time"=>"2021-06-24T13:31:20.134+03:00",
     "end_time"=>"2021-06-25T13:31:20.134+03:00",
     "active"=>true,
     "packet_type"=>"temporary_view_1d",
     "prolongable"=>true,
     "russian_region"=>"Region 1",
     "hidden"=>false,
     "price"=>nil,
     "fias_id"=>"00000000-0000-0000-0000-000000000000",
     "prices"=>
      [{"code"=>"job_publish", "price"=>0},
       {"code"=>"super_job_publish", "price"=>0},
       {"code"=>"ultra_job_publish", "price"=>0},
       {"code"=>"job_refresh", "price"=>0},
       {"code"=>"job_highlight", "price"=>0},
       {"code"=>"contact_view", "price"=>0},
       {"code"=>"job_elevate_plus", "price"=>0},
       {"code"=>"job_elevate_1d", "price"=>0},
       {"code"=>"job_elevate_3d", "price"=>0}],
     "limits"=>
      [{"code"=>"job_publish", "initial"=>0, "left"=>0, "default"=>0},
       {"code"=>"super_job_publish", "initial"=>0, "left"=>0, "default"=>0},
       {"code"=>"ultra_job_publish", "initial"=>0, "left"=>0, "default"=>0},
       {"code"=>"job_refresh", "initial"=>0, "left"=>0, "default"=>0},
       {"code"=>"job_highlight", "initial"=>0, "left"=>0, "default"=>0},
       {"code"=>"contact_view", "initial"=>250, "left"=>249, "default"=>150},
       {"code"=>"job_elevate_plus", "initial"=>0, "left"=>0, "default"=>0},
       {"code"=>"job_elevate_1d", "initial"=>0, "left"=>0, "default"=>0},
       {"code"=>"job_elevate_3d", "initial"=>0, "left"=>0, "default"=>0}],
     "created_at"=>"2021-06-25T13:21:20.559+03:00",
     "initial_views_limit"=>250,
     "views_limit"=>249,
     "buy_transaction_type"=>"PaymentTransaction",
     "job_category_uuid"=>"00000000-0000-0000-0000-000000000000"},
   "last_step"=>
    {"start_time"=>"2021-06-25T13:31:20.134+03:00",
     "end_time"=>"2021-06-26T13:31:20.134+03:00",
     "active"=>false,
     "packet_type"=>"temporary_view_1d",
     "prolongable"=>true,
     "russian_region"=>"Region 1",
     "hidden"=>false,
     "price"=>nil,
     "fias_id"=>"00000000-0000-0000-0000-000000000000",
     "prices"=>
      [{"code"=>"job_publish", "price"=>0},
       {"code"=>"super_job_publish", "price"=>0},
       {"code"=>"ultra_job_publish", "price"=>0},
       {"code"=>"job_refresh", "price"=>0},
       {"code"=>"job_highlight", "price"=>0},
       {"code"=>"contact_view", "price"=>0},
       {"code"=>"job_elevate_plus", "price"=>0},
       {"code"=>"job_elevate_1d", "price"=>0},
       {"code"=>"job_elevate_3d", "price"=>0}],
     "limits"=>
      [{"code"=>"job_publish", "initial"=>0, "left"=>0, "default"=>0},
       {"code"=>"super_job_publish", "initial"=>0, "left"=>0, "default"=>0},
       {"code"=>"ultra_job_publish", "initial"=>0, "left"=>0, "default"=>0},
       {"code"=>"job_refresh", "initial"=>0, "left"=>0, "default"=>0},
       {"code"=>"job_highlight", "initial"=>0, "left"=>0, "default"=>0},
       {"code"=>"contact_view", "initial"=>250, "left"=>250, "default"=>150},
       {"code"=>"job_elevate_plus", "initial"=>0, "left"=>0, "default"=>0},
       {"code"=>"job_elevate_1d", "initial"=>0, "left"=>0, "default"=>0},
       {"code"=>"job_elevate_3d", "initial"=>0, "left"=>0, "default"=>0}],
     "created_at"=>"2021-06-25T13:21:20.628+03:00",
     "initial_views_limit"=>250,
     "views_limit"=>250,
     "buy_transaction_type"=>"PaymentTransaction",
     "job_category_uuid"=>"00000000-0000-0000-0000-000000000000"},
   "days_count"=>2,
   "left_days_count"=>1,
   "id"=>141,
   "free"=>true,
   "auto_prolongable"=>false},
 "meta"=>{"hidden_packet_expires_at"=>nil}}
```


