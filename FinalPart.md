## Глава дополнительная. Рефлексируем на тему полученных знаний и навыков 



Какие проблемы 

Не решенные 
Меня смущает, что пользователь фактически напрямую обращается к мидлваре, т.е. он может запускать код который общается с сервером. Возможно попробовать запускать сначала экшен, а в нем уже отправлять обработку (передавать по очереди) в мидлваре, но это противоречит тому, что говорит Дэн Абрамов о экшенах

Решенные
1. Наводим красоту в подключении
2. Наводим красоту в сообщених

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
