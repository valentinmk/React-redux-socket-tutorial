# Часть дополнительная. Рефлекс(ируем)

## Экшены клиента и экшены мидлвара

В настоящей реализации все работает и это неоспоримо хорошо. Но как показал случай с подключением, иногда мы не знаем чем закончится действие с нашей серверной частью.

Давайте тогда наведем порядок с подключением.
Идея очень простая - давайте разделять серверные и клиентские экшены (просто на уровне нашего понимания и на уровне проектирования)

У нас в настоящий момент
`SOCKETS_CONNECTING` - клиентский экшен, который записывает в редукс информацию о текущем подключении и дополняет массив истории
`SOCKETS_CONNECT` - это событие нашего мидлвара, которое запускает и подключение к сокетам и вызывает клиентское событие.
Улучшаем наше решение - добавляем экшен `SOCKETS_CONNECTED`, который будет нам сообщать о том, что подключение на самом деле выполнено.
Абсолютно аналогичным образом давайте поступим в экшеном отключения.

Добавляем экшены
```js
export const SOCKETS_CONNECTING = 'SOCKETS_CONNECTING';
export const SOCKETS_CONNECT = 'SOCKETS_CONNECT';
export const SOCKETS_CONNECTED = 'SOCKETS_CONNECTED';
export const SOCKETS_DISCONNECTING = 'SOCKETS_DISCONNECTING';
export const SOCKETS_DISCONNECT = 'SOCKETS_DISCONNECT';
export const SOCKETS_DISCONNECTED = 'SOCKETS_DISCONNECTED';
```
Добавляем эшен функции

```js
export function socketsConnecting() {
  return {type: SOCKETS_CONNECTING};
}
export function socketsConnect() {
  return {type: SOCKETS_CONNECT};
}
export function socketsConnected() {
  return {type: SOCKETS_CONNECTED};
}
export function socketsDisconnecting() {
  return {type: SOCKETS_DISCONNECTING};
}
export function socketsDisconnect() {
  return {type: SOCKETS_DISCONNECT};
}
export function socketsDisconnected() {
  return {type: SOCKETS_DISCONNECTED};
}
```
А теперь рассказываем редюсеру в чем заключается разница между нашими экшенами

```js
case SOCKETS_CONNECTING:
  return Object.assign({}, state, {
    loaded: true,
    message: 'Connecting...',
    connected: false,
    history: [
      ...state.history,
      {
        loaded: true,
        message: 'Connecting...',
        connected: false
      }
    ]
  });
case SOCKETS_CONNECTED:
  return Object.assign({}, state, {
    loaded: true,
    message: 'Connected. All system online.',
    connected: true,
    history: [
      ...state.history,
      {
        loaded: true,
        message: 'Connecting...',
        connected: false
      }
    ]
  });
case SOCKETS_DISCONNECTING:
  return Object.assign({}, state, {
    loaded: true,
    message: 'Disconnecting...',
    connected: true,
    history: [
      ...state.history,
      {
        loaded: true,
        message: 'Disconnecting...',
        connected: true
      }
    ]
  });
case SOCKETS_DISCONNECTED:
  return Object.assign({}, state, {
    loaded: true,
    message: 'Disconnected',
    connected: false,
    history: [
      ...state.history,
      {
        loaded: true,
        message: 'Disconnected',
        connected: false
      }
    ]
  });
```
Теперь давайте расскажем мидлвару как запускать данные события.

```js
case 'SOCKETS_CONNECT':
  if (socketExample !== null) {
    console.log('SOCKETS_DISCONNECTING');
    store.dispatch(SocketExampleActions.socketsDisconnecting());
    socket.close();
    store.dispatch(SocketExampleActions.socketsDisconnected());
  }
  console.log('SOCKETS_CONNECTING');
  store.dispatch(SocketExampleActions.socketsConnecting());
  socketExample = new WebSocket('wss://echo.websocket.org/');
  socketExample.onmessage = onMessage(socketExample, store);
  socketExample.onclose = onClose(store);
  socketExample.onopen = onOpen(store, action.token);
  break;
case 'SOCKETS_DISCONNECT':
  if (socketExample !== null) {
    console.log('SOCKETS_DISCONNECTING');
    store.dispatch(SocketExampleActions.socketsDisconnecting());
    socketExample.close();
  }
  socketExample = null;
  store.dispatch(SocketExampleActions.socketsDisconnected());
  break;
```

Окей, что интересного тут произошло - мы изменили функцию `onOpen` передали в нее стор, чтобы вызывать экшены (см. ниже). Мы добавили в места, которые взаимодействуют с сокетами экшены так, как видится правильны.

```js
  const onOpen = (store, token) => evt => {
    console.log('WS is onOpen');
    console.log('token ' + token);
    console.log('evt ' + evt.data);
    store.dispatch(SocketExampleActions.socketsConnected());
  };
```
Проверяем.


### Косяки
дисконектинг переводит в тру (хотя подключение не переводилось в тру)
сам дисконектинг работает.




## Какие проблемы

### Не решенные
Меня смущает, что пользователь фактически напрямую обращается к мидлваре, т.е. он может запускать код который общается с сервером. Возможно попробовать запускать сначала экшен, а в нем уже отправлять обработку (передавать по очереди) в мидлваре, но это противоречит тому, что говорит Дэн Абрамов о экшенах

### Решенные
1. Наводим красоту в подключении
2. Наводим красоту в сообщениях

На будущее (см. "Проблема с отключением от сервера в случае его недоступности или сброса подключения")
По аналогии с красотой в подключении имеет смысл держать различные экшены для результатов обработки ... или универсальные экшены для обработки результатов события.


### Проблема с отключением от сервера в случае его недоступности или сброса подключения



```js
  const onOpen = (token) => evt => {
    console.log('WebSocket is onOpen');
    console.log('token ' + token);
    console.log('evt ' + evt.data);
  };

  const onClose = (store) => evt => {
    store.dispatch(SocketExampleActions.socketsDisconnecting());
    console.log('WebSocket is onClose');
    console.log('evt ' + evt.data);
    if (socketExample !== null){
      console.log('socket termination');
      socketExample = null;
    }
  };

  return store => next => action => {
    switch (action.type) {
      case 'SOCKETS_CONNECT':
        console.log('socketExample = ' + socketExample);
        if (socketExample !== null) {
          console.log('SOCKETS_DISCONNECTING');
          store.dispatch(SocketExampleActions.socketsDisconnecting());
          socketExample.close();
        }
        console.log('SOCKETS_CONNECTING');
        socketExample = new WebSocket('ws://echo.websocket.org/');
        store.dispatch(SocketExampleActions.socketsConnecting());
        socketExample.onclose = onClose(store);
        socketExample.onopen = onOpen(action.token);
        break;
      case 'SOCKETS_DISCONNECT':
        if (socketExample !== null) {
          console.log('SOCKETS_DISCONNECTING');
          socketExample.close();
        }

        break;
      default:
        return next(action);
    }
```
