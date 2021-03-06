# UTON-HACK-2.0

![logo](./resources/logo.png)

`Biteacon - это проект, зародившийся в рамках UTON HACK 2.0. Мы очень ценим данные и знания, а еще больше ценим когда ими 
 умеют правильно пользоваться! Поэтому наш проект представляет из себя 2 пользовательских решения:`
* [biteacon.xyz](http://biteacon.xyz) - business intelligence web application, работающее с проиндексированной базой данных
блокчейна LikeLib. Позволяет очень удобно проводить аналитику блокчейна.
* [biteacon bot](https://t.me/biteacon_bot) - телеграм-бот, реализующий функционал блокчейн-эксплорера для LikeLib.

`Также обратите внимание на наши другие ресурсы:`
* [biteacon channel](https://t.me/biteacon) - телеграм канал, где вы можете узнать о функциональных возможностях нашего продукта
и быть в курсе обновлений и новых событий.
* [biteacon GitHub](https://github.com/biteacon) - наша организация на GitHub, там вы всегда сможете найти самые актуальные
версии нашего проекта.

## Content
* [Content](#Content)
* [Biteacon BI](#Biteacon-BI)
* [Biteacon bot](#Biteacon-bot)
* [Architecture](#Architecture)
* [Installation and Configuration](#Installation-and-Configuration)
* [Repositories](#Repositories)

## Biteacon BI

## Biteacon bot

## Architecture
### Solution architecture

Этапы работы программного решения:

1) Разворачивается LikeLib blockchain.
2) Разворачивается база данных Indexed LikeLib, Telegram bot DB и GraphQL Hasura.
3) Кроулер подключается к блокчейну LikeLib и к базе данных Indexed LikeLib, после чего начинает вытягивать блок один за другим и записывать 
их в проиндексированную базу данных.
4) Разворачивается инфраструктура Apache Superset.
    1) Поднимается Application DB для Apache Superset.
    2) Поднимается Redis для хэширования запросов Apache Superset.
    3) Поднимается сервер Apache Superset и пользовательский интерфейс.
    4) Происходит инициализация базовой конфигурации.
    5) Apache Superset подключается к Indexed LikeLib.
5) Разворачивается телеграм-бот.
6) Пользователь использует телеграм-бота для удовлетворения задач по поиску и исследованию, а Superset BI восполняет
потребности в аналитических данных. 


![architecture](./resources/Architectures-Biteacon.png)
* Crawler - `Особенностью записи данных кроулером в Indexed LikeLib database является порция данных размером в 1 блок. Каждый блок
 и все его транзакции, адреса, должны записываться атомарно. При этом возникает сложность в получении всеобъемлющего 
 набора данных для атомарной записи, поскольку LikeLib fullnode не позволяет по http получить многие необходимые связи. 
 Мы решили эту проблему засчет хэша внутри кроулера и механизма матчинга.`
* Telegram bot - `это LikeLib blockchain explorer прямо в мессенджере. Одной из его особенностей является возможность 
как выполнять заданные команды, так и выполнять поисковые запросы. Сложность поисковых функций может быть заметна на 
примере хэша транзакций или хэша блока. Из-за их кодировки, хэши в отличие от адресов, не могут использоваться как
полноценные ссылки и поиск по ключу требует валидации многих дополнительных сценариев. Также факт наличия команд и 
поисковых запросов требует наличия отдельного оркестратора команд.`
<br></br><br></br>
### Indexed LikeLib schema

`Данная схема позволяет в полной мере делать гибкие выборки, в отличие от key-value, поскольку прописаны все возможные 
связи и проиндексированы значемые столбцы. Теперь становится возможным увидеть транзакции по блоку, все виды транзакций 
в аккаунтах(minging, input, output) и многое другое!`

![likelib](./resources/Architectures-Indexed-LikeLib.png)
<br></br><br></br>
### Telegram-bot schema
Телеграм бот нуждается в отдельной схеме базы данных по двум причинам:
1) Добавление команд в избранное
2) Регистрация(инвайты) в Biteacon(Superset BI) через механизмы бота.

![bot](./resources/Architectures-Telegram-bot-DB.png)
<br></br><br></br>
## Installation and Configuration
`Почти все компоненты нашего решения используют Docker. Следуя инструкциям в каждом отдельном репозитории, вы можете 
быстро и легко развернуть рабочий стенд.`

`P.s. Перед запуском настоятельно рекомендуется поменять пароли!`
## Repositories
#### Актуальные
* [biteacon/UTON-HACK-2.0](https://github.com/biteacon/UTON-HACK-2.0) - репозиторий корневой документации по проекту Biteacon
* [biteacon/telegram-bot](https://github.com/biteacon/telegram-bot) - исходный код телеграм бота(эксплорер LikeLib), написанного на java
* [biteacon/crawler-likelib](https://github.com/biteacon/crawler-likelib) - кроулер, выкачивающий все данные из блокчейна LikeLib 
с хранением данных в key-value базе данных по http. Преобразует полученные данные и записывает в реляционную схему(postgres-hasura)
* [biteacon/postgres-hasura](https://github.com/biteacon/postgres-hasura) - исходный код конфигурации базы данных Postgres и 
Hasura GraphQL сервера, содержит миграции
* [biteacon/incubator-superset](https://github.com/biteacon/incubator-superset) - (Fork) Apache Superset (incubating) is a 
modern, enterprise-ready business intelligence web application
#### Устаревшие
`За время проведения хакатона пришлось изменить стратегию и архитектуру проекта. Мы пришли к выводу, что демонстрация
распределенных JOIN'ов из нескольких разных базах данных, с использованием Presto, потребует большей подготовки. 
Сложность оказалась в Superset, поскольку его компонент SQLAlchemy не поддерживает распределенный JOIN разных баз данных.`

`P.s. Вы все же можете использовать нашу готовую конфигурацию, чтобы увидеть распределенные запросы. Вы можете поднять
 в докере базу данных(postgres-hasura), а потом запустить в докере presto-server. После этого можно 
 подключиться через presto-cli и отправлять распределенные запросы.`
* [biteacon/presto-server](https://github.com/biteacon/presto-server) - сконфигурированный однонодовый сервер распределенного 
кластера запросов
* [biteacon/presto-cli](https://github.com/biteacon/presto-cli) - клиент для подключения к ноде кластера