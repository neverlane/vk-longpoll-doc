# User Long Poll API

### План документации:
0. [Отказ от ответственности](#отказ-от-ответственности)
1. [Подключение](#подключение)
2. [Возвращаемые ошибки](#возвращаемые-ошибки)
3. [Получение истории событий](#получение-истории-событий)
4. [Структура сообщения](#структура-сообщения)
5. [Описание событий](#описание-событий)
   - [Событие 2. Установка флагов сообщения](#событие-2-установка-флагов-сообщения)
   - [Событие 3. Сброс флагов сообщения](#событие-3-сброс-флагов-сообщения)
   - [Событие 4. Новое сообщение](#событие-4-новое-сообщение)
   - [Событие 5. Редактирование сообщения](#событие-5-редактирование-сообщения)
   - [Событие 6. Прочтение входящих сообщений](#событие-6-прочтение-входящих-сообщений)
   - [Событие 7. Прочтение исходящих сообщений](#событие-7-прочтение-исходящих-сообщений)
   - [Событие 8. Друг появился в сети](#событие-8-друг-появился-в-сети)
   - [Событие 9. Друг вышел из сети](#событие-9-друг-вышел-из-сети)
   - [Событие 10. Сброс флагов беседы](#событие-10-сброс-флагов-беседы)
   - [Событие 12. Установка флагов беседы](#событие-12-установка-флагов-беседы)
   - [Событие 13. Удаление всех сообщений в диалоге](#событие-13-удаление-всех-сообщений-в-диалоге)
   - [Событие 18. Добавление сниппета к сообщению](#событие-18-добавление-сниппета-к-сообщению)
   - [Событие 19. Сброс кеша сообщения](#событие-19-сброс-кеша-сообщения)
   - [Событие 21. Неизвестно](#событие-21-неизвестно)
   - [Событие 51. Изменение данных чата (не следует использовать)](#событие-51-изменение-данных-чата-не-следует-использовать)
   - [Событие 52. Изменение данных чата](#событие-52-изменение-данных-чата)
   - [Событие 63. Статус набора сообщения](#событие-63-статус-набора-сообщения)
   - [Событие 64. Статус записи голосового сообщения](#событие-64-статус-записи-голосового-сообщения)
   - [Событие 80. Изменение количества непрочитанных диалогов](#событие-80-изменение-количества-непрочитанных-диалогов)
   - [Событие 81. Изменение состояния невидимки друга](#событие-81-изменение-состояния-невидимки-друга)
   - [Событие 114. Изменение настроек пуш-уведомлений в беседе](#событие-114-изменение-настроек-пуш-уведомлений-в-беседе)
   - [Событие 115. Звонок](#событие-115-звонок)
   - [Событие 119. Ответ callback-кнопки](#событие-119-ответ-callback-кнопки)
6. [Дополнительные данные](#дополнительные-данные)
   - [Флаги сообщений](#флаги-сообщений)
   - [Сервисные сообщения](#сервисные-сообщения)
   - [Клавиатура для ботов](#клавиатура-для-ботов)
   - [Вложения](#вложения)
   - [Зачем нужен random_id](#зачем-нужен-random_id)

Документация написана для __10__ версии Long Poll.

Последняя версия Long Poll - __13__, но я не рекомендую использовать версии выше __10__, так как они не содержат полезных изменений.

## Отказ от ответственности

Автор не несет ответственности за точность, полноту или качество предоставленной информации. Используйте последующую информацию на свой страх и риск.

[Правила пользования сайтом](https://vk.com/terms), пункт 6.7:

> 6.7. Создаваемые Пользователями приложения API должны использовать **только опубликованные на Сайте методы API**, а также ID, защищенный ключ и сервисный ключ доступа, указанные в настройках данных приложений. Использование других методов API, а также ID, защищенного ключа и сервисного ключа доступа приложений API третьих лиц, в т.ч. приложения API Администрации Сайта, строго запрещено. Пользователь обязуется регулярно проверять перечень разрешённых методов и незамедлительно вносить корректировки в функциональность своих приложений API в соответствии с изменениями перечня. **За нарушение настоящего пункта Пользователь несет предусмотренную применимым законодательством, настоящими Правилами и иными документами Администрации Сайта ответственность**. Администрация Сайта при этом оставляет за собой право на защиту собственных прав и законных интересов.

## Подключение

__Long Polling__ - это технология, используемая для получения событий в реальном времени, которая основана на бесконечной цепочке запрос - ответ - запрос... При подаче запроса сервер возвращает ответ не сразу, а когда придет новое событие или истечет время ожидания. Подробнее про данную технологию можно прочитать [здесь](https://learn.javascript.ru/long-polling).

Ссылка для запроса составляется следующим образом:

> https://**`server`**?act=a_check&key=**`key`**&ts=**`ts`**&wait=**`wait`**&mode=**`mode`**&version=**`version`**

* `server`, `key` и `ts` получаются методом [`messages.getLongPollServer`](https://vk.com/dev/messages.getLongPollServer)
* `version` - Версия LongPoll
* `wait` - Время ожидания нового события в секундах, максимум `90`, рекомендую от `10` до `25`
* `mode` - Дополнительные опции ответа:
  * `2` - Возвращать вложения
  * `8` - Возвращать данные в [114](#событие-114-изменение-настроек-пуш-уведомлений-в-беседе) и [115](#событие-115-звонок) событиях
  * `32` - Возвращать `pts`
  * `64` - Возвращать данные о платформе в событии [онлайна друга](#событие-8-друг-появился-в-сети)
  * `128` - Возвращать [`random_id`](#зачем-нужен-random_id)

Настоятельно рекомендую использовать в `mode` все возможные опции.

В `Node.js` ссылку можно составить следующим образом:
```js
// server, key и ts нужно получить заранее с помощью метода messages.getLongPollServer.
const link = `https://${server}?` + require('querystring').stringify({
  act: 'a_check',
  key: key,
  ts: ts,
  wait: 10,
  mode: 2 | 8 | 32 | 64 | 128,
  version: 10
});
```

После выполнения запроса сервер вернет ответ следующего вида:
```ts
interface LongPollResult {
  // Приходит, если нет ошибок или при failed: 1
  ts?: number
  // Приходит, если нет ошибок и при наличии флага 32
  pts?: number
  // Приходит, если нет ошибок
  updates?: any[]
  failed?: 1 | 2 | 3 | 4
  // Приходят только при failed: 4
  min_version?: 0
  max_version?: 13
}
```

После обработки ответа для продолжения работы Long Poll нужно повторить запрос, перед этим заменив `ts` и `pts` на новый из ответа.

## Возвращаемые ошибки

Иногда вместо поля `updates` в ответе может прийти поле `failed`. Чаще всего приходит `2` ошибка, но могут прийти и другие:

1. Устарела история событий. Решается [получением истории событий](#получение-истории-событий) и использованием переданного `ts` далее.
2. Истекло время действия ключа. Решается получением нового `key` из метода [`messages.getLongPollServer`](https://vk.com/dev/messages.getLongPollServer).
3. Информация о пользователе утрачена. Решается [получением истории событий](#получение-истории-событий) и получением `key` и `ts` из метода [`messages.getLongPollServer`](https://vk.com/dev/messages.getLongPollServer).
4. Передана неправильная версия. В ответе приходят поля `min_version` и `max_version`, которые обозначают минимальную и максимальную версию.

## Получение истории событий

Чтобы получить историю событий, нам необходим `pts`, получение которого включается полем `need_pts: 1` в [`messages.getLongPollServer`](https://vk.com/dev/messages.getLongPollServer) и добавлением в `mode` флага `32` при [выполнении запроса](#подключение).

Для получения истории мы будем использовать метод [`messages.getLongPollHistory`](https://vk.com/dev/messages.getLongPollHistory) с указанием следующих параметров:
- `pts` - Последний полученный `pts` из LongPoll
- `msgs_limit` - Максимальное количество передаваемых сообщений. Минимум `200`, рекомендую `500`, максимум `1000`
- `onlines` - `1` если возвращать события `8` и `9` ([онлайн](#событие-8-друг-появился-в-сети) и [оффлайн](#событие-9-друг-вышел-из-сети) друга), `0` если нет
- `lp_version` - Версия LongPoll
- `fields` - Поля [пользователей и групп](https://vk.com/dev/objects/user), которые могут прийти вместе с историей

Ответ выглядит следующим образом:
```ts
// Все приведенные типы:
// https://github.com/danyadev/vk-desktop/tree/typescript/src/types
interface LongPollHistoryResult {
  // Здесь содержатся некоторые события из LongPoll,
  // но вся информация действительно является только числами:
  // [event_id, msg_id, flag, peer_id] или что-то похожее
  history: number[][]
  from_pts: number
  new_pts: number
  conversations: VKConversation[]
  messages: {
    count: number
    items: VKMessage[]
  }
  profiles?: VKUser[]
  groups?: VKGroup[]
  more?: true
}
```

Поле `history` идентично полю `updates`, которое приходит из Long Poll, за исключением удаления некоторых лишних событий (т.к. приходит просто измененный объект беседы или сообщения) и сокращения некоторых событий:
- `3` - [Сброс флагов сообщения](#событие-3-сброс-флагов-сообщения)
- `4` - [Новое сообщение](#событие-4-новое-сообщение)
- `5` - [Редактирование сообщения](#событие-5-редактирование-сообщения)
- `18` - [Добавление сниппета к сообщению](#событие-18-добавление-сниппета-к-сообщению)

Структура этих событий выглядит так (все с типом `number`):
```
[event_id, msg_id, flags, peer_id]
```

Всю информацию о сообщениях и беседах можно взять из полей `messages` и `conversations`, где содержатся данные из API.

Если в ответе придет поле `more`, то после обработки всех событий нужно будет повторить запрос, указав в поле `pts` пришедший `new_pts`.

## Структура сообщения

Структура сообщения встречается сразу в 4 событиях ([`3`](#событие-3-сброс-флагов-сообщения), [`4`](#событие-4-новое-сообщение), [`5`](#событие-5-редактирование-сообщения) и [`18`](#событие-18-добавление-сниппета-к-сообщению)), поэтому было решено описать ее отдельно.

В данной структуре есть еще несколько структур, которые описаны в других частях документации:

- [Флаги сообщений](#флаги-сообщений)
- [Шаблоны](https://vk.com/dev/bot_docs_templates?f=5.%2B%D0%A8%D0%B0%D0%B1%D0%BB%D0%BE%D0%BD%D1%8B%2B%D1%81%D0%BE%D0%BE%D0%B1%D1%89%D0%B5%D0%BD%D0%B8%D0%B9)
- [Клавиатура для ботов](#клавиатура-для-ботов)
- [Сервисные сообщения](#сервисные-сообщения)
- [Вложения](#вложения)
- [random_id](#зачем-нужен-random_id)

```ts
[
  event_id: 3 | 4 | 5 | 18,
  msg_id: number,
  flags: number,
  peer_id: number,
  timestamp: number,
  text: string,

  {
    // Приходит только в лс | в каналах
    title?: ' ... ' | ''
    // Наличие emoji
    emoji?: '1'
    // id автора сообщения. Приходит только в беседах
    from?: string
    // Наличие шаблона (для получения шаблона нужно получить сообщение из API)
    has_template?: '1'
    // 1: Список упомянутых людей прямо или через @online; Ответ на сообщение от [user_id]; @all
    // 2: Исчезающее сообщение в обычном чате
    marked_users?: [
      | [1, number[] | 'all']
      | [2, 'all']
    ]
    // Клавиатура для ботов (в т.ч. инлайн)
    keyboard?: MessageKeyboard
    // Количество секунд до исчезновения сообщения в обычном чате
    expire_ttl?: string
    // Количество секунд до исчезновения сообщения в фантомном чате
    ttl?: number
    // Сообщение исчезло, приходит в 18 событии
    is_expired?: '1'
    // Сервисное сообщение
    ...action
  },

  {
    // Есть пересланное сообщение или ответ на сообщение
    fwd?: '0_0'
    // Ответ на сообщение: '{"conversation_message_id":number}'
    reply?: string

    // Количество вложений в поле attachments
    // Число в строке
    attachments_count?: string
    // JSON с массивом вложений
    // Приходит только для некоторых типов вложений
    attachments?: string

    // Описание вложений вида { attach1_type, attach1, ... }
    // Все значения здесь имеют тип строки (и могут быть объектом в строке по типу '{"id":1}')
    ...attachments
  },

  // Возвращается, если в mode есть флаг 128
  random_id: number,
  // id сообщения относительно беседы
  conversation_message_id: number,
  // 0 (не редактировалось) или timestamp (время редактирования)
  edit_time: number
]
```

Стоит отметить, что в поле `text` переносы строк обозначаются как `<br>`, а символы `"`, `&`, `<` и `>` экранируются.

## Описание событий

### Событие 2. Установка флагов сообщения

Возможные значения [флагов сообщения](#флаги-сообщений):
1. Пометка важным (`8 important`)
2. Пометка как спам (`64 spam`)
3. Удаление сообщения (`128 deleted`)
4. Удаление для всех (`128 deleted` и `131072 deleted_all`)
5. Прослушивание голосового сообщения (`4096 audio_listened`)

В случае прослушивания голосового сообщения событие приходит как при прослушивании собеседником
вашего голосового сообщения, так и при прослушивании вами голосового сообщения собеседника.

Вручную прослушать голосовое сообщение собеседника можно с помощью метода `messages.markAsListened` с параметром `message_id`.
Метод вернет `1` при первом прослушивании голосового сообщения, а `0` при последующих или при попытке прослушать свое сообщение.

```ts
//                    [msg_id, flags,  peer_id]
type LongPollEvent2 = [number, number, number];
```

### Событие 3. Сброс флагов сообщения

Возможные значения [флагов сообщения](#флаги-сообщений):
1. Прочитано сообщение (`1 unread`). Не рекомендую использовать, потому что иногда этот флаг не приходит
2. Отмена пометки важным (`8 important`)
3. Отмена пометки сообщения как спам (`64 spam` и `32768 cancel_spam`)
4. Восстановление удаленного сообщения (`128 deleted`)

В 3 и 4 случаях возвращается [сообщение](#структура-сообщения).

```ts
//                    [msg_id, flags,  peer_id]
type LongPollEvent3 = [number, number, number] | LongPollMessage;
```

### Событие 4. Новое сообщение

Данное событие возвращает новое [сообщение](#структура-сообщения).

```ts
type LongPollEvent4 = LongPollMessage;
```

### Событие 5. Редактирование сообщения

Данное событие возвращает отредактированное [сообщение](#структура-сообщения).

```ts
type LongPollEvent5 = LongPollMessage;
```

### Событие 6. Прочтение входящих сообщений

Вы прочитали в диалоге `peer_id` сообщения до `msg_id` включительно.  
`count` - количество непрочитанных в диалоге сообщений.

```ts
//                    [peer_id, msg_id, count]
type LongPollEvent6 = [number,  number, number];
```

### Событие 7. Прочтение исходящих сообщений

Собеседник прочитал в диалоге `peer_id` сообщения до `msg_id` включительно.  
`count` - количество __ваших__ непрочитанных сообщений в диалоге.

```ts
//                    [peer_id, msg_id, count]
type LongPollEvent7 = [number,  number, number];
```

### Событие 8. Друг появился в сети

Структура: `[-user_id, platform, timestamp, app_id]`
- `user_id` - `id` друга
- `platform` - Тип устройства, с которого онлайн друг:
   - `1` - `m.vk.com` или неизвестное мобильное приложение
   - `2` - iPhone
   - `3` - iPad
   - `4` - Android
   - `5` - windows Phone
   - `6` - Windows 8
   - `7` - `vk.com` или неизвестное десктопное приложение
- `timestamp` - Время онлайна в секундах
- `app_id` - `id` приложения, с которого онлайн друг

```ts
//                    [-user_id, platform, timestamp, app_id, 0 | 1, 0]
type LongPollEvent8 = [number,   number,   number,    number, 0 | 1, 0];
```

### Событие 9. Друг вышел из сети

Структура: `[-user_id, isTimeout, timestamp, app_id]`
- `user_id` - `id` друга
- `isTimeout` - `1` если бездействовал 5 минут, `0` если покинул сайт
- `timestamp` - Время наступления оффлайна в секундах
- `app_id` - `id` приложения, с которого был онлайн друг

```ts
//                    [-user_id, isTimeout, timestamp, app_id, 0, 0]
type LongPollEvent9 = [number,   0 | 1,     number,    number, 0, 0];
```

### Событие 10. Сброс флагов беседы

Приходит при следующих событиях: (+ приходящий флаг)
1. Просмотрено упоминание (ответ на сообщение, пуш или @all) - `17408` (`1024` + `16384`)
2. Просмотрено исчезающее сообщение - `17408` (`1024` + `16384`)
3. Беседа была отмечена прочитанной - `1048576`

Кроме этих событий есть еще одно, но пока неясно, что именно произошло. Его флаг - `537919488` (`1048576`, `536870912`).

Так же это событие приходит когда удаляются все сообщения, которые добавляют этот флаг.

Событие означает удаление у беседы соответствующего значка, то есть:
 - для 1 и 2 пункта событие придет только при просмотре всех сообщений такого рода
 - для 3 пункта событие придет только при прочтении сообщений после ручной пометки беседы как непрочитанной

```ts
//                     [peer_id, flags]
type LongPollEvent10 = [number,  number];
```

### Событие 12. Установка флагов беседы

Приходит при следующих событиях: (+ приходящий флаг)
1. Появилось упоминание (ответ на сообщение, пуш или @all) - `16384`
2. Появилось исчезающее сообщение - `16384`
3. Беседа была отмечена непрочитанной - `1048576`

Событие означает добавление к беседе соответствующего значка, то есть:
 - для 1 и 2 пункта событие придет только для первого сообщения в таком роде
 - для 3 пункта событие придет только при самостоятельной пометке беседы как непрочитанной

```ts
//                     [peer_id, flags]
type LongPollEvent12 = [number,  number];
```

### Событие 13. Удаление всех сообщений в диалоге

В беседе `peer_id` были удалены все сообщения до `last_msg_id` включительно.

```ts
//                     [peer_id, last_msg_id]
type LongPollEvent13 = [number,  number];
```

### Событие 18. Добавление сниппета к сообщению

Приходит при следующих событиях:
1. Добавился сниппет (ссылка) - к вложениям добавляется `link`.
2. Сообщение исчезло - удаляется текст и все вложения, добавляется ключ `is_expired: true`.
3. Пришел перевод голосового сообщения.

В общем это событие работает как редактирование сообщения со стороны ВКонтакте,
так что обрабатывать его нужно так же, как и обычное редактирование сообщения.

Данное событие возвращает [сообщение](#структура-сообщения).

```ts
type LongPollEvent18 = LongPollMessage;
```

### Событие 19. Сброс кеша сообщения

Приходит в двух случаях:
1. По какой-либо причине изменилось сообщение (без явного редактирования). Необходимо переполучить сообщение через API.
2. Сообщение исчезло. В данном случае можно игнорировать это событие, если вы обработали исчезновение сообщения в 18 событии.

__ВАЖНО__: если вы собираетесь обрабатывать исчезновение сообщений, то я рекомендую игнорировать 19 событие и обрабатывать
только 18 событие. Однако сначала приходит 19 событие, а потом уже 18, поэтому нужно сначала дождаться, пока обработается
18 событие и там к сообщению добавится поле `is_expired`, и только затем здесь проверять это сообщение на наличие этого поля.
Дождаться обработки можно либо с помощью таймаутов, а можно воспользоваться событиями об окончании обработки события
(в зависимости от реализации).

```ts
//                     [msg_id]
type LongPollEvent19 = [number];
```

### Событие 21. Неизвестно

```ts
//                     [peer_id, msg_id]
type LongPollEvent21 = [number,  number];
```

### Событие 51. Изменение данных чата (не следует использовать)

Событие означает, что в беседе `chat_id` изменились какие-то данные.
Более подробно все расписано в [52 событии](#событие-52-изменение-данных-чата).

```ts
//                     [chat_id]
type LongPollEvent51 = [number];
```

### Событие 52. Изменение данных чата

Структура: `[type, peer_id, extra]`
- `peer_id` - `id` беседы, в которой произошло событие
- `extra` - Дополнительная информация, которая зависит от `type`
- `type` - Тип действия:
   - `1` - Изменилось название беседы
      - Основная информация приходит в [сервисном сообщении](#сервисные-сообщения) в [4 событии](#событие-4-новое-сообщение)
      - `extra` = `0`
   - `2` - Обновилась аватарка беседы
      - Основная информация приходит в [сервисном сообщении](#сервисные-сообщения) в [4 событии](#событие-4-новое-сообщение)
      - `extra` = `0`
   - `3` - Назначен новый администратор
      - `extra` = `id` нового администратора
   - `4` - Изменение прав в беседе  
      Поле `extra` является маской, которая может включать следующие флаги:
      - `1` - Приглашать участников в беседу могут только администраторы
      - `2` - Неизвестно
      - `4` - Менять закрепленное сообщение могут только администраторы
      - `8` - Редактировать информацию беседы (фото, название) могут только администраторы
      - `16` - Добавлять администраторов кроме создателя могут и другие администраторы
   - `5` - Закрепление или открепление сообщения
      - `extra` = `conversation_message_id` сообщения при закреплении и `0` при откреплении
   - `6` - Вступление в беседу (по ссылке, приглашение или возвращение в беседу)
      - `extra` = `id` добавленного юзера
   - `7` - Выход из беседы
      - `extra` = `id` вышедшего из беседы
   - `8` - Исключение из беседы
      - `extra` = `id` исключенного из беседы
   - `9` - Разжалован администратор
      - `extra` = `id` разжалованного из администраторов
   - `11` - Появление и скрытие клавиатуры
      - `extra` = `id` диалога, в котором произошло действие
   - `19` - Начало и окончание группового звонка
      - `extra` = `1` при начале, а `0` при конце звонка
      - Может приходить несколько раз для одного и того же звонка с `extra` = `1`

```ts
//                     [type,   peer_id, extra]
type LongPollEvent52 = [number, number,  number];
```

### Событие 63. Статус набора сообщения

Означает, что в беседе `peer_id` начали писать текст `from_ids_count` людей. Их `id` записаны в `from_ids`.

В массиве `from_ids` иногда может появиться и ваш `id`, так что нужно проверять и удалять его из списка.

```ts
//                     [peer_id, from_ids[], from_ids_count, timestamp]
type LongPollEvent63 = [number,  number[],   number,         number];
```

### Событие 64. Статус записи голосового сообщения

Идентичен [событию 63](#событие-63-статус-набора-сообщения), но вызывается в случае записи голосового сообщения, а не при наборе текста.

### Событие 80. Изменение количества непрочитанных диалогов

Структура: `[count, count_with_notifications]`
- `count` - Количество непрочитанных диалогов
- `count_with_notifications` - Количество непрочитанных сообщений со включенными уведомлениями

```ts
//                     [count,  count_with_notifications, 0, 0]
type LongPollEvent80 = [number, number,                   0, 0];
```

### Событие 81. Изменение состояния невидимки друга

Структура: `[-user_id, state, timestamp]`
- `user_id` - `id` друга
- `state` - `1` если включена, `0` если выключена
- `timestamp` - время последнего онлайна друга

```ts
//                     [-user_id, state,  timestamp, -1, 0]
type LongPollEvent81 = [number,   number, number,    -1, 0];
```

### Событие 114. Изменение настроек пуш-уведомлений в беседе

Структура: `[{ peer_id, sound, disabled_until }]`
- `peer_id` - `id` беседы, в которой включили или выключили уведомления
- `sound` - нестабильный параметр, не рекомендую его обрабатывать
- `disabled_until` может быть трех видов:
   - `0` - Уведомления включены
   - `-1` - Уведомления выключены
   - `number, > 0`, - Уведомления выключены до определенного времени, где `number` - дата включения уведомлений

```ts
type LongPollEvent114 = [{
  peer_id: number
  sound: 0 | 1
  disabled_until: 0 | -1 | number
}];
```

### Событие 115. Звонок

¯\\\_(ツ)\_/¯

### Событие 119. Ответ callback-кнопки

[Callback-кнопки](https://vk.com/dev/bots_docs_5?f=4.4.+Callback-кнопки) работают следующим образом:

1. Бот отправляет клавиатуру (обычную или инлайн), где находится callback-кнопка;
2. Пользователь нажимает на эту кнопку и клиент вызывает метод `messages.sendMessageEvent`
с указанием `peer_id` и `message_id`. Метод же возвращает строку, которая является `event_id`;
3. Бот получает событие `message_event` и вызывает метод [messages.sendMessageEventAnswer](https://vk.com/dev/messages.sendMessageEventAnswer);
4. Этот метод вызывает `119` событие LongPoll у пользователя, где в `action` прописано действие, которое необходимо выполнить клиенту.

После получения `event_id` на 2 стадии нужно начать ждать `119` событие LongPoll с нужным `event_id`.
Если за минуту бот так и не пришлет событие, то ожидание ответа следует прекратить.

```ts
type LongPollEvent119 = [{
  // Отрицательный ID бота, который отвечает на клик по кнопке
  owner_id: number
  // ID беседы, в которой находится и бот, и пользователь
  peer_id: number
  // Уникальный ID события, действующий 1 минуту.
  // Нужен для идентификации кликнутой кнопки
  event_id: string
  // Не приходит, если не нужно выполнять никакое действие
  // т.е. бот отправил пустой payload или неизвестный тип действия
  action?:
    // Показать снекбар с текстом `text`
    | { type: 'show_snackbar', text: string }
    // Открыть ссылку `link`
    | { type: 'open_link', link: string }
    // Открыть приложение по ссылке
    // https://vk.com/app${app_id}_${owner_id}#${hash}
    // https://vk.com/app${app_id}_${owner_id} (если hash = '')
    // https://vk.com/app${app_id} (если owner_id = undefined и hash = '')
    | { type: 'open_app', app_id: number, owner_id?: number, hash: string }
}];
```

## Дополнительные данные

### Флаги сообщений

Маской называют сумму некоторых флагов (степеней двойки), которую можно использовать как хорошую замену для объектов или массивов.

```js
// Обычно маску записывают подобным образом:
const mask = 1 | 2 | 8 | 64 | 1024;
// Но можно воспользоваться и обычным сложением:
const mask = 1 + 2 + 8 + 64 + 1024;
// А еще можно не писать большие цифры, а записывать степени двойки в таком формате:
const mask = 1 << 11 | 1 << 15;
```

Для того, чтобы не запутаться в больших значениях степеней двойки,
можно использовать `1 << n`, что означает `2` в степени `n`.

| Название       | Описание                                 | Значение  | Как число |
| :------------: | :--------------------------------------: | :-------: | :-------: |
| unread         | Непрочитанное сообщение                  | `1 << 0`  | `1`       |
| outbox         | Исходящее сообщение                      | `1 << 1`  | `2`       |
| important      | Важное сообщение                         | `1 << 3`  | `8`       |
| chat           | Отправка сообщения в беседу через vk.com | `1 << 4`  | `16`      |
| friends        | Исходящее; входящее от друга в лс        | `1 << 5`  | `32`      |
| spam           | Пометка сообщения как спам               | `1 << 6`  | `64`      |
| deleted        | Удаление сообщения локально              | `1 << 7`  | `128`     |
| audio_listened | Прослушано голосовое сообщение           | `1 << 12` | `4096`    |
| chat2          | Отправка в беседу через клиенты          | `1 << 13` | `8192`    |
| cancel_spam    | Отмена пометки как спам                  | `1 << 15` | `32768`   |
| hidden         | Приветственное сообщение от группы       | `1 << 16` | `65536`   |
| deleted_all    | Удаление сообщения для всех              | `1 << 17` | `131072`  |
| chat_in        | Входящее сообщение в беседе              | `1 << 19` | `524288`  |
| silent         | "Бесшумное" сообщение (без уведомления)  | `1 << 20` | `1048576` |
| reply_msg      | Ответ на сообщение                       | `1 << 21` | `2097152` |

Бесшумное сообщение можно отправить, добавив к параметрам метода [`messages.send`](https://vk.com/dev/messages.send) ключ `silent: true`:
- Такое сообщение отправится без уведомления пользователю, даже если у него включены уведомления
- Упоминание или ответ на сообщение тоже не отправит уведомления
- Сообщение о выходе из беседы автоматически приходит без уведомления

Пример определения наличия флага в маске:
```js
const mask = 1 | 2 | 32; // = 35

8 & mask // вернет 0 (false)
2 & mask // вернет 2 (true)
```

### Сервисные сообщения

Сервисное сообщение описывается ключом `source_act` и ключами с дополнительными данными.

Возможные значения `source_act`:
- __`chat_create`__ - Создание беседы
   - `source_text` - Название беседы
- __`chat_photo_update`__ - Обновление фотографии беседы
   - Нет дополнительных ключей, но добавляет фотографию во вложениях
- __`chat_photo_remove`__ - Удаление фотографии беседы
   - Нет дополнительных ключей
- __`chat_title_update`__ - Обновление названия беседы
   - `source_old_text` - Старое название беседы (но при получении сообщения из API его не будет)
   - `source_text` - Новое название беседы
- __`chat_pin_message`__ - Закрепление сообщения
   - `source_mid` - `id` закрепившего сообщение
   - `source_message` - обрезанное закрепляемое сообщение
   - `source_chat_local_id` - `conversation_message_id` закрепляемого сообщения
- __`chat_unpin_message`__ - Открепление сообщения
   - `source_mid` - `id` открепившего сообщение
   - `source_chat_local_id` - `conversation_message_id` открепляемого сообщения
- __`chat_invite_user`__ - Вступление или возвращение в беседу
   - `source_mid` - `id` вступившего или вернувшегося
   - Если `source_mid` совпадает с `from`, то он вернулся сам, иначе - его пригласили
- __`chat_invite_user_by_link`__ - Вступление в беседу по ссылке
   - Нет дополнительных ключей
- __`chat_kick_user`__ - Исключение или выход из беседы
   - `source_mid` - `id` исключенного или вышедшего
   - если `source_mid` совпадает с `from`, то он вышел сам, иначе - его исключили
- __`chat_screenshot`__ - Создание скриншота с фантомным сообщением
   - `source_mid` - `id` создавшего скриншот
- __`chat_group_call_started`__ - Начало группового звонка в беседе __`deprecated`__
   - __Больше не приходит в LongPoll__
   - Вместо этого сервисного сообщения создается сообщение с вложением `group_call_in_progress`
   - Приходит для старых сообщений через API, но только при получении через токен VK для Android
- __`chat_invite_user_by_call`__ - Приглашение пользователя в звонок
   - `source_mid` - `id` приглашенного пользователя
- __`chat_invite_user_by_call_join_link`__ - Присоединение пользователя к звонку по ссылке
   - Нет дополнительных ключей

Пример описания сервисного сообщения:
```js
{
  source_act: 'chat_pin_message',
  source_mid: '88262293',
  source_message: 'Сообщение, которое будет в закрепе',
  source_chat_local_id: '5517'
}
```

### Клавиатура для ботов

Клавиатура для ботов представляет собой объект с описанием ее типа и кнопок.
Основная структура представлена ниже, остальную информацию можно узнать в [документации](https://vk.com/dev/bots_docs_3?f=4.2.%20%D0%A1%D1%82%D1%80%D1%83%D0%BA%D1%82%D1%83%D1%80%D0%B0%20%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85).
```ts
// Типы у объекта клавиатуры:
// https://github.com/danyadev/vk-desktop/blob/typescript/src/types/VKKeyboard.ts
interface LongPollKeyboard {
  // Клавиатура для сообщения или для беседы
  inline?: true
  // Скрывать ли клавиатуру при клике на кнопку (не работает для inline)
  one_time: boolean
  buttons?: VKKeyboardButton[][]
}
```

### Вложения

Список известных на данный момент вложений: `geo`, `doc`, `link`, `poll`, `wall`, `call`, `gift`, `story`, `photo`, `audio`, `video`, `event`, `market`, `artist`, `sticker`, `article`, `podcast`, `graffiti`, `wall_reply`, `audio_message`, `money_request`, `audio_playlist`, `group_call_in_progress`.

Однако названия вложений, полученных через LongPoll, могут не совпадать с теми, что приходят через API:
- `event`, приходящий в API, в LongPoll обозначается как `group`
- `audio_message` и `graffiti` из LongPoll обозначаются как `doc`, но при этом добавляется ключ `attach*_kind` в ответе вложения со значением `audiomsg` или `graffiti`
- `artist`, `audio_playlist` и `article`, которые приходят в LongPoll, через API отображаются как `link`

Вложение `geo` (прикрепленное местоположение) является еще одним исключением и приходит не как `attach*`, а просто как ключи `geo` и иногда `geo_provider`. Также при получении сообщения через [`messages.geyById`](https://vk.com/dev/messages.getById) ключ `geo` будет находиться не во вложениях, а в "корне" сообщения.

Сообщение с вложением `group_call_in_progress` создается, когда пользователь начинает групповой звонок. При окончании группового звонка
сообщение с этим вложением удаляется и создается новое сообщение с вложением `call`.

На данный момент вложения передаются достаточно неудобным для обработки способом. Вот пример вложений, состоящих из фотографии, документа и аудиозаписи:
```js
{
  attach1: '88262293_457290160',
  attach1_type: 'photo',
  attach2: '88262293_532324610',
  attach2_type: 'doc',
  attach3: '88262293_535133534',
  attach3_kind: 'audiomsg',
  attach3_type: 'doc'
}
```

__Хорошие новости:__ ВКонтакте уже начал улучшать возвращение вложений и теперь некоторые вложения возвращаются в таком же виде, как и в API. На данный момент я заметил это только для стикеров и голосовых сообщений, но возможно в будущем все вложения будут приходить в подобном виде.

Для того, чтобы вытащить их названия, нужно писать дополнительный код. Однако вытаскивать идентификаторы вложений нет смысла, потому что они не представляют никакой ценности. Чтобы получить нормальную информацию о вложениях, нужно получить сообщение через API, используя метод [`messages.geyById`](https://vk.com/dev/messages.getById). Пример кода для получения массива с названиями вложений на `JavaScript`:
```js
function getAttachments(data) {
  const attachments = [];

  if (data.geo) {
   attachments.push({ type: 'geo' });
  }

  for (const key in data) {
    const match = key.match(/attach(\d+)$/);

    if (match) {
      const id = match[1];
      const kind = data[`attach${id}_kind`];
      let type = data[`attach${id}_type`];

      if (kind === 'audiomsg') type = 'audio_message';
      if (kind === 'graffiti') type = 'graffiti';
      if (type === 'group') type = 'event';

      attachments.push({ type });
    }
  }

  return attachments;
}
```

При работе с вложениями можно попробовать найти необходимый элемент в [документации](https://vk.com/dev/objects), однако у некоторых вложений документация не обновлена или вовсе отсутствует.

### Зачем нужен random_id

Начнем с того, что `random_id` начиная с версии API `5.90` стал обязательным параметром.
Но это не означает, что всем нужно генерировать уникальные значения для этого параметра.
Если вы его не собираетесь использовать, то можете указать в его значении число `0`.

Значение `random_id` должно быть уникальным в течение __одного часа__ в рамках `app_id`, `id` пользователя и `peer_id` диалога.

Значение `random_id` может принимать числа от `-2147483648` (`-(2^31)`) до `2147483647` (`2^31 - 1`).
Если число будет больше или меньше данного порога, то из переданного числа отнимется этот лимит.

В документации написано, что `random_id` необходим для того, чтобы предотвратить повторную отправку одинакового сообщения.
Однако это не единственное возможное применение данного параметра.

Наверняка вы уже видели, что во всех мессенджерах ВКонтакте при отправке сообщения само сообщение отображается сразу,
но около сообщения некоторое время видно иконку часов. Эта иконка означает, что сообщение еще не пришло обратно через LongPoll.

Чтобы реализовать подобную фичу, нужно определить, какое именно сообщение, приходящее из LongPoll, было только что отправлено.
Для этого нужно:
1) Создать `random_id` и сохранить его в списке с загружаемыми сообщениями
2) Отправить сообщение
3) Дождаться прихода из LongPoll сообщения с нашим `random_id`
4) Удалить `random_id` из списка, тем самым пометив сообщение как загруженное.
