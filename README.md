# Задание

Форма регистрации с отправкой email после одобрения из внешней системы. Дана форма регистрации в нашем приложении, в которой необходимо заполнить:

- Имя
- Фамилия
- Адрес электронной почты
- День рождения
- Город проживания
- Город регистрации

После отправки формы, мы регистрируем данные из нее в нашей БД, а также отправляем ее для одобрения во внешней системе. Пусть обмен с этой внешней системой будет через RabbitMQ. После одобрения или отклонения заявки, наше приложение должно отправить сообщение на электронную почту нашему пользователю с результатом проверки.

## Tech stack
- Kotlin
- Spring boot / Ktor 
- Message Broker(RabbitMQ, Kafka), предпочтительнее Kafka
- DB: PostgreSQL, MongoDB. 
- Тесты - Kotlin Test, KSpec, Spek
Остальное по вкусу. 

## Набросок архитектуры

![arch][img/arch.png]

###Опциональная задача
каждому запросу в сервис одобрения, нужно прикладывать уникальный ID. Генерацию ID  реализовать в свободной форме в отдельном сервисе. 
 
### Рекомендуемый Data Flow 

- User делает запрос в API сервис, у которого 2 endpoint’a: 
  - POST api/send-verification: В теле запроса - данные формы, ответ - id-запроса в систему одобрения
  - GET api/check-verification-status: ответ - статус обращения(одобрено, не одобрено)
- (Опционально) API-сервис получает уникальный ID из специального сервиса-генератора.
- Собирается модель-DTO и отправляется в Approver Service. 
- Aprover Service может являться прокси между какой-то внешней рекомендательной системой и дальнейшей бизнес-логикой или может быть ею самой, в виде заглушки для этой внешней системы. Тут все на ваше усмотрение.
- Как приходит ответ из внешней системы(или заглушки) - он отправляется в Message Broker(RabbitMQ, Kafka)
- Mailer-сервис подписан(consumes) на обновления из Aprover Service, смотрит данные, которые ему пришли, и отсылает определенное письмо на почту пользователя(Одобрено, не одобрено)
 
### Рекомендации: 

Все сервисы обернуть в docker-образы и сделать docker-compose.yaml для запуска окружения
Если используете Spring boot - используйте Spring Cloud стэк - Consul Service Discovery, Hystrix, Zipkin и.т.п.
Задание можно привести в разную степень готовности - как в production ready, так и в PoC(Proof of concept). Делайте как считаете нужным, но постарайтесь не потратить более 10-15 часов. 


