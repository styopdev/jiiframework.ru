Комет-клиент
=======

На клиенте немного проще: есть один компонент `Jii.comet.client.Client`, который подключается к одному из серверов и
обменивается данными.

```js
application: {
    components: {
        comet: {
            className: 'Jii.comet.client.Client',
            serverUrl: 'http://localhost:4401/comet',
            autoOpen: true
        },
        
        // ...
    }
}
```

Полный список свойств, методов и событий представлен ниже.

### Jii.comet.client.Client

#### Свойства
- `transport` (object) - компонент или его конфигурация, имплементирующий интерфейс
`Jii.comet.client.transport.TransportInterface` для обмена сообщениями с сервером.
- `plugins` (object) - набор компонентов или их конфигурация, каждый из которых имплементирует интерфейс
`Jii.comet.client.plugin.PluginInterface` для расширения возможностей комет клиента.
- `workersCount` (number|null) - максимальное количество воркеров сервера. Эта настройка используется для балансировки
нагрузки на клиенте. По-умолчанию, `null` - отключена.
- `autoOpen` (boolean) - если указано `true`, то при инициализации приложения комет клиент автоматически будет
подключаться к серверу. По-умолчанию, `true`.

#### События
- `open` - Событие запускается при успешном открытии соединения.
- `close` - Событие запускается при закрытии соединения.
- `beforeSend` - Событие запускается при любом исходящем сообщении. Первым аргументом обработчика будет передан
экземпляр класса `Jii.comet.client.MessageEvent` с параметром `message`. Вы можете изменить отправляемое сообщение, изменяя
параметр `message`, на сервер отправятся данные из параметра `message`.
- `channel` - Событие запускается при любом входящем сообщении в любой подписанный канал. Первым агрументом обработчика
будет передан экземпляр класса `Jii.comet.ChannelEvent` с параметрами `channel` (string) и `message` (string).
- `channel:%my_channel_name%` - Событие запускается при любом входящем сообщении в подписанный канал, указанный после `:`.
Например, при вызове `Jii.app.comet.on('channel:test', ...)` происходит подписка на сообщения в канал `test`. Как и выше,
в обработчик события первым аргументом будет передан экземпляр класса `Jii.comet.ChannelEvent` с параметрами
`channel` (string) и `message` (string).
- `message` - Событие запускается при любом входящем сообщении. Первым агрументом обработчика будет передан экземпляр
класса `Jii.comet.server.MessageEvent` с параметром `message` (string).
- `beforeRequest` - Событие запускается перед тем, как клиент пытается отправить на сервер запрос на выполнения действия.
Первым аргументом обработчика будет передан экземпляр класса `Jii.comet.client.RequestEvent` с параметрами `route` (string)
и `params` (object). Вы можете изменить параметры запроса в этом событии, на сервер отправятся данные из параметра `params`.
- `request` - Событие запускается при ответе сервера на запрошенное ранее действие. Первым аргументом обработчика будет
передан экземпляр класса `Jii.comet.client.RequestEvent` с параметрами `route` (string) и `params` (object).

## Плагины

Плагины позволяют расширять возможности комет клиента. По-умолчанию, установлен плагин для автоматического
восстановления соединения `Jii.comet.client.plugin.AutoReconnect`.
Плагин очень прост в создании и подключении. Интерфейс плагина `Jii.comet.client.plugin.PluginInterface` не требует
имплементации методов, а лишь предоставляет доступ к комет-клиенту через параметр `comet`, через который можно
подписываться на события комета. Устанавливается и настраивается плагин через конфигурацию, путем добавления его
в секцию `plugins`:

```js
application: {
    components: {
        comet: {
            className: 'Jii.comet.client.Client',
            // ...
            plugins: {
                autoReconnect: {
                    enable: false
                },
                myPlugin: {
			        className: 'app.components.MyCometPlugin'
                }
            }
        },
        
        // ...
    }
}
```