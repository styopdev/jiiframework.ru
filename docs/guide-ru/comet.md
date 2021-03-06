Комет
=======

Для начала немного справки о том, что такое комет ([wiki](https://ru.wikipedia.org/wiki/Comet_(%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5))):

> Comet (в веб-разработке) — любая модель работы веб-приложения, при которой постоянное HTTP-соединение позволяет
> веб-серверу отправлять (push) данные браузеру без дополнительного запроса со стороны браузера.

Jii-comet - это масштабируемый, готовый к высоким нагрузкам и плохому интернету транспорт, реализующий постоянную связь
между клиентом и сервером для мгновенного обмена данными.

Jii-comet предоставляет набор компонентов и классов, которые упрощают обмен сообщениями между каналами, подписки на них,
обмена данными между серверами и так далее. Сам модуль не умеет доставлять сообщения на клиент и обратно, но в нем
заложена абстракция, чтобы это можно было делать любой из существующих популярных библиотек (например,
[socket.io](http://socket.io/), [sockjs](https://github.com/sockjs/sockjs-client)), а так же чтобы это было надежно и масштибаруемо.

## Возможности

Клиент:
- Отправка и прием сообщений из каналов
- Подписка на каналы
- Вызов действий на сервере
- Балансировка (случайный выбор сервера)
- Возможность смены транспорта сообщений

Сервер:
- Отправка и прием сообщений из каналов
- Подписка на сообщения из канала
- Подписывание соединения на канал
- Отправка сообщений в соединение
- Возможность смены транспорта сообщений, хаба и очереди
- Масштабирование на несколько процессов и серверов
- Режим listen-only (прослушивание каналов)
- Запуск действий из очереди

## Установка

Комет устанавливается как [компонент приложения](structure-application-components), на сервере и клиенте соответственно.
На сервере комет открывает и "слушает" данные из указанного порта, а клиент при инициализации приложения подключается
к этому порту и ждет информацию.

Рассмотрим пример приложения, где клиент подписывается на сервере на канал `test` и отправляет в него сообщение.
Сервер:

```js
var Jii = require('jii');
require('jii-comet');

require('jii-workers')
    .application('comet', Jii.mergeConfigs(
        {
            application: {
                basePath: __dirname,
                components: {
                    comet: {
                        className: 'Jii.comet.server.Server',
                        port: 4401,

                        /**
                         *
                         * @param {Jii.comet.server.ConnectionEvent} event
                         */
                        'on addConnection': function(event) {
                            Jii.app.comet.subscribe(event.connection.id, 'test');
                        }
                    }
                }
            }
        },
        custom.comet || {}
    ));
```

Клиент:

```js
require('jii/deps');
require('jii-comet/sockjs');

Jii.createWebApplication({
    application: {
        basePath: location.href,
        components: {
            comet: {
                className: 'Jii.comet.client.Client',
                serverUrl: 'http://localhost:4401/comet',
                                                        
                /**
                 *
                 * @param {Jii.base.Event} event
                 */
                'on open': function(event) {
                    Jii.app.comet.send('test', {message: 'Hello World'});
                },
                                                        
                /**
                 *
                 * @param {Jii.comet.ChannelEvent} event
                 */
                'on channel:test': function(event) {
                    console.log('Income message: ' + event.params.message);
                }
            }
        }
    }
}).start();
```

Подробнее о конфигурации смотрите в следующих разделах: [комет-сервер](comet-server) и [комет-клиент](comet-client).
