Комет-сервер
=======

Jii-comet предоставляет два класса для создания сервера:
- `Jii.comet.server.HubServer` - сервер, настроенный только для прослушивания сообщений от соединений, при этом самих
подключений у него нет, но он может отправить данные в подключение, находящееся в соседнем процессе. Т.е. в такой сервер
напрямую (на порт) невозможно подключиться, однако сообщение можно отправить через другие процессы (к которым подключены
клиенты).
- `Jii.comet.server.Server` - полноценный сервер, унаследован он предыдущего. Хранит и поддерживает прямое соединение
с клиентами, может обмениваться данными с клиентом.

Такое резделение необходимо для гибкого масштабирования приложения. Например, вы можете создать несколько процессов
с компонентом `Jii.comet.server.Server`, которые будут только поддерживать соединение с клиентами, но не обрабатывать
бизнес логику. И создать несколько процессов `Jii.comet.server.HubServer`, которые будут выполнять тяжелые вычесления.
Это позволит сделать комет-канал более отзывчивым: если бы все операции были бы на одном сервере, то сложные операции
затормаживали бы процесс, и все последующие запросы (даже легкие) были бы с задержкой.

Если у вас нет больших нагрузок или вы не знаете что выбрать, выбирайте компонент `Jii.comet.server.Server`, так как он
содержит в себе весь функционал.

Пример конфигурации для сервера выглядит так:

```js
application: {
    components: {
        comet: {
            className: 'Jii.comet.server.Server',
            host: '0.0.0.0',
            port: 3100
        },
        
        // ...
    }
}
```

Полный список свойств, методов и событий представлен ниже.

### Jii.comet.server.HubServer

#### Свойства
- `listenActions` (boolean) - принимать ли запросы на вызов действий от клиентов. По-умолчанию, `true`.
- `hub` (object) - хаб, компонент или его конфигурация, имплементирующий интерфейс `Jii.comet.server.hub.HubInterface` для
обмена сообщениями между процессами или серверами. По-умолчанию создается компонент `Jii.comet.server.hub.Redis`.
- `queue` (object) - очередь, компонент или его конфигурация, имплементирующий интерфейс `Jii.comet.server.queue.QueueInterface`
для накопления очереди вызовов действий (actions). По-умолчанию создается компонент `Jii.comet.server.queue.Redis`.

#### Методы
- `start()` - открывает соединение для компонентов `hub` и `queue`, начинает слушать сообщения из каналов.
- `stop()` - разрывает все соединения компонентов и перестает слушать информацию из вне.
- `sendToChannel(channel, data)` - отправляет сообщение в канал в рамках хаба. Здесь `channel` - (string) название
канала, а `data` - (string|object|*) данные для отправки.
- `sendToConnection(id, data)` - как и предыдущий метод, отправляет данные, но не в канал, а напрямую в соединение.
Здесь `id` (string) - это идентификатор соединения, взятый из `Jii.comet.server.Connection`, а `data`
- (string|object|*) данные для отправки.
- `on(name, handler, data, isAppend)` и `off(name, handler)` - методы для подписки и отписки на события.
Список событий описан ниже, описание агрументов можно найти в [разделе событий](concept-events).
- `hasChannelHandlers(name)` - Вовращает `true`, если на канал `name` (string) есть подписчики в рамках данного процесса.

#### События
- `channel` - Событие запускается при любом входящем сообщении в любой канал. Первым агрументом обработчика будет передан
экземпляр класса `Jii.comet.ChannelEvent` с параметрами `channel` (string) и `message` (string).
- `channel:%my_channel_name%` - Событие запускается при любом входящем сообщении в канал, указанный после `:`.
Например, при вызове `Jii.app.comet.on('channel:test', ...)` происходит подписка на сообщения в канал `test`. Как и выше,
в обработчик события первым аргументом будет передан экземпляр класса `Jii.comet.ChannelEvent` с параметрами
`channel` (string) и `message` (string).
- `message` - Событие запускается при любом входящем сообщении в хаб. Первым агрументом обработчика будет передан
экземпляр класса `Jii.comet.server.MessageEvent` с параметром `message` (string).

### Jii.comet.server.Server
Наследует вышеперечисленные свойства, методы и события и добавляет следующие:

#### Свойства
- `host` (string) - хост, ожидающий входящие соединения от клиентов. По-умолчанию, `0.0.0.0`.
- `port` (number) - порт для входящих соединений. По-умолчанию, `4100`.
- `transport` (object) - компонент или его конфигурация, имплементирующий интерфейс `Jii.comet.server.transport.TransportInterface` для
обмена сообщениями непосредственно с клиентами (браузером).

#### Методы
- `subscribe(connectionId, channel)` и `unsubscribe(connectionId, channel)` - подписывает и отписывает соединение
на заданный канал. Jii рекомендует делать подписку на канал именно на сервере, чтобы избежать гонки сообщений.
В методах `connectionId` (string) - это идентификатор соединения, взятый из `Jii.comet.server.Connection`,
а `channel` (string) - имя канала.

#### События
- `addConnection` - Событие возникает при присоединении нового клиента. Первым агрументом обработчика будет передан
экземпляр класса `Jii.comet.server.ConnectionEvent` с параметром `connection` (`Jii.comet.server.Connection`).
- `removeConnection` - Событие возникает при потере соединения с клиентом. Аналогично предыдущему, первым агрументом
обработчика будет передан экземпляр класса `Jii.comet.server.ConnectionEvent`.

### Jii.comet.server.Connection
Содержит информацию о соединении с клиентом

#### Свойства
- `id` (string) - идентификатор соединения. Используется для отправки сообщений напрямую в соединение.
- `request` (`Jii.comet.server.Request`) - информация о соединении: заголовки, ip адрес, порт подключения и так далее.
- `originalConnection` (object) - внутренний экземпляр соединения, полученный от транспорта. Специфичен для каждого транспорта.