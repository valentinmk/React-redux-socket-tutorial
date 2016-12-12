# Мидлваре

Более подробно о мидлваре в редюксе можно почитать на официальном сайте [http://redux.js.org/docs/advanced/Middleware.html](http://redux.js.org/docs/advanced/Middleware.html).

А еще вот та самая статья [https://exec64.co.uk/blog/websockets_with_redux/](https://exec64.co.uk/blog/websockets_with_redux/).

Давайте создадим наш собственный мидлваре, в котором будем реализовывать интерфейс к сервису построенному на websockets.

###Первый проход

Создаем новый файл `./src/redux/middleware/socketExampleMiddleware.js`

В этот файл нам нужно добавить экшены, которыми мы будем манипулировать. По своему принципу мидлваре напоминает структуру редюсера, но этому будет проиллюстрировано ниже. 

Для начала просто проверяем, что концепция работает и делаем тестовый прототип, который будет подтверждением подхода.

 

```js
import * as socketExampleActions from 'redux/modules/socketexamplemodule';

export default function createSocketExampleMiddleware() {
  let socketExample = null;
  socketExample = true;
  socketExampleActions();

  return store => next => action => {
    switch (action.type) {
      default:
        console.log(store, socketExample, action);
        return next(action);
    }
  };
}
```
Подробнее. Вообще мидлваре управляет самим редуксом и как он обрабатывает события и состояния внутри себя. Использую конструкцию `return store => next => action =>` мы вмешиваемся в каждый экшен происходящий в редукци и по полю `switch (action.type)` выполняем те или иные действия.

У нас сейчас действительно простой пример и логирование в консоль самый просто способ посмотреть, что у нас прилитает в переменных `store, socketExample, action`. (`socketExampleActions();` оставили просто, чтобы не ругался валидатор, вообще они нам понадобятся).

Не проверяем, у нас ничего не работает, потому что мы не подключили наш класс в мидлваре. Исправляем. 

В файле `./src/redux/create.js` меняем пару строк.

```js
import createSocketExampleMiddleware from './middleware/socketExampleMiddleware';
//...
  const middleware = [
    createMiddleware(client),
    reduxRouterMiddleware,
    thunk,
    createSocketExampleMiddleware()
  ];
```

Запускаем проверяем. Теперь в консоле полный беспорядок, но оно же целом все работает!
Коммит: 

### Второй проход

Ну теперь давайте подключимся к websockets. Здесь и далее даются пробные вырианты, которые иллюстрируют ход раработки и могут содержать ошибки. 

Добавляем в файл `src\redux\middleware\SocketExampleMiddleware.js` функции, которыми будем обрабатывать события подключения и отключения.

```js
  const onOpen = (token) => evt => {
    console.log('WS is onOpen');
    console.log('token ' + token);
    console.log('evt ' + evt.data);
  };

  const onClose = () => evt => {
    console.log('WS is onClose');
    console.log('evt ' + evt.data);
  };
```
добавляем наши редюсеры и убираем лишнее логирование.

```js
      case 'SOCKETS_CONNECT':
        if (socketExample !== null) {
          console.log('SOCKETS_DISCONNECTING');
          store.dispatch(SocketExampleActions.socketsDisconnecting());
          socket.close();
        }
        console.log('SOCKETS_CONNECTING');
        socketExample = new WebSocket('ws://echo.websocket.org/');
        store.dispatch(SocketExampleActions.socketsConnecting());
        socketExample.onclose = onClose();
        socketExample.onopen = onOpen(action.token);
        break;
      default:
        return next(action);
```
Начинем разбираться и понимает, что сейчас ничего не работает. Мы ловим экшен `SOCKETS_CONNECT`, которого у нас пока нет. Но у нас есть другой экшен `SOCKETS_CONNECTING`, отлично будем его использовать  меняем скрипт. 

```js
      case 'SOCKETS_CONNECTING':
        if (socketExample !== null) {
          console.log('SOCKETS_DISCONNECTING');
          store.dispatch(SocketExampleActions.socketsDisconnecting());
          socket.close();
        }
        console.log('SOCKETS_CONNECTING');
        socketExample = new WebSocket('ws://echo.websocket.org/');
        store.dispatch(SocketExampleActions.socketsConnecting());
        socketExample.onclose = onClose();
        socketExample.onopen = onOpen(action.token);
        break;
      default:
        return next(action);
```


> !!! Внимание после этого скрипт будет находиться в бесконечном цикле - сохраните все или не запускайте на данном этапе страничку.



Проверяем и видим, что все пошло не так. В консоле постоянные `SOCKETS_CONNECTING` и `SOCKETS_DISCONNECTING`. Закрываем вкладку или браузер. 
Что происходит: мидлваре "слушает" стор на предмет экшенов `store => next => action =>` и включается в обработку если находит свой экшен. Выход простой - экшены для мидлеваре должны быть всегда отдельными от экшенов, которые происходят на стороне клиентов.

Как быть дальше.

Наш вариант (я думаю он не один) будет таким: 
* Пользователь будет вызывать нажатием кнопки экшены мидлвара
* Который будет вызывать уже "интерфейсные" экшены. 

Что на практике будет означать 
* `SOCKETS_CONNECT` вызывается пользователем
* при его обработки будет вызываться `SOCKETS_CONNECTING`
* который будет уже обновлять стор и соответствующим образом представлять действие на стороне клиента.

Давайте исправим все это.

Во-первых нам не хватает экшенов.

Дополняем наши 2 экшена новыми в файле `src\redux\modules\socketexamplemodule.js`.

```js
export const SOCKETS_CONNECTING = 'SOCKETS_CONNECTING';
export const SOCKETS_CONNECT = 'SOCKETS_CONNECT';
export const SOCKETS_DISCONNECTING = 'SOCKETS_DISCONNECTING';
export const SOCKETS_DISCONNECT = 'SOCKETS_DISCONNECT';
```
И объявим функции они нам пригодятся.

```js
export function socketsConnecting() {
  return {type: SOCKETS_CONNECTING};
}
export function socketsConnect() {
  return {type: SOCKETS_CONNECT};
}
export function socketsDisconnecting() {
  return {type: SOCKETS_DISCONNECTING};
}
export function socketsDisconnect() {
  return {type: SOCKETS_DISCONNECT};
}
```

Теперь нужно дать возможность пользователю запускать данные действия. По идеи нужно лезть в `src\components\SocketExampleComponents\SocketConnectionLog.jsр`, но на самом деле управляющие функции ему передают через компонент react js, так что правим только `src\containers\SocketExample\SocketExamplePage.js`. 

```js
  static propTypes = {
    loaded: PropTypes.bool,
    message: PropTypes.string,
    connected: PropTypes.bool,
    history: PropTypes.array,
    socketsConnecting: PropTypes.func,
    socketsDisconnecting: PropTypes.func,
// Here
    socketsConnect: PropTypes.func,
    socketsDisconnect: PropTypes.func
  }
  render() {
// Here
    const {loaded, message, connected, socketsConnect, socketsDisconnect, history} = this.props;
    return (
      <div className="container">
        <h1>Socket Example Page</h1>
        <Helmet title="Socket Exapmle Page"/>
        <p>Sockets not connected</p>
        <SocketConnectionLog
          loaded={loaded}
          message={message}
          connected={connected}
// And here
          connectAction={socketsConnect}
          disconnectAction={socketsDisconnect}
          history={history}/>
        <SocketMessageLog />
      </div>
    );
  }
```
Можно попробовать, но будьте осторожны, оно действительно работате и хуже того браузер опять ушел в цикл. 
Возвращаемся к `src\redux\middleware\SocketExampleMiddleware.js` и наводим порядок. 

Убираем (стираем строку), она нам больше не нужна:

```js
  socketExample = true;
```

Изменяем один кейс
```js
      case 'SOCKETS_CONNECT':
```

Добавляем кейс на обработку отключения:

```js
      case 'SOCKETS_DISCONNECT':
        if (socketExample !== null) {
          console.log('SOCKETS_DISCONNECTING');
          store.dispatch(SocketExampleActions.socketsDisconnecting());
          socketExample.close();
        }
        socketExample = null;
        break;
```

Для правильного отображения событий disconnect передаем в системное событие также функцию экшена и там его вызываем.

```js
socketExample.onclose = onClose(store);
```

изменяем сам обработчик
```js
  const onClose = (store) => evt => {
    console.log('WebSocket is onClose');
    console.log('evt ' + evt.data);
    store.dispatch(SocketExampleActions.socketsDisconnect());
  };
```

Ну вроде все - проверяем. Проверяем как работает. Видим в закладке нетворк подключения и отключения на вебсокеты. 

Если у вас не работает подключение особенно в случае недоступности или сброса подключения добро пожаловать в раздел "Проблема с отключением от сервера в случае его недоступности или сброса подключения" [Последней части](https://valentinmk.gitbooks.io/react-redux-socket-tutorial/content/FinalPart.html).

Коммитимся.

### Проход второй. Делаем сообщения

Все аналогично 
Добавляем экшены в `src\redux\modules\socketexamplemodule.js`

```js
export const SOCKETS_MESSAGE_SENDING = 'SOCKETS_MESSAGE_SENDING';
export const SOCKETS_MESSAGE_SEND = 'SOCKETS_MESSAGE_SEND';
export const SOCKETS_MESSAGE_RECEIVING = 'SOCKETS_MESSAGE_RECEIVING';
export const SOCKETS_MESSAGE_RECEIVE = 'SOCKETS_MESSAGE_RECEIVE';
```

Добавляем обработчики
```js
export function socketsMessageSending(sendMessage) {
  return {type: SOCKETS_MESSAGE_SENDING, message_send: sendMessage};
}
export function socketsMessageSend(sendMessage) {
  return {type: SOCKETS_MESSAGE_SEND, message_send: sendMessage};
}
export function socketsMessageReceiving(receiveMessage) {
  return {type: SOCKETS_MESSAGE_RECEIVING, message_receive: receiveMessage};
}
```
Нам, на самом деле, не нужна обработка socketsMessageReceive, потому что пользователю не нужно вмешиваться в процесс получения сообщения.

Ну и занимаемся магией обработки в файле `src\redux\middleware\SocketExampleMiddleware.js` - все как обычно ничего необычного.
В нашем обработчике получаем сообщение и передаем его в редукс.
```js
  const onMessage = (ws, store) => evt => {
    // Parse the JSON message received on the websocket
    const msg = evt.data;
    store.dispatch(SocketExampleActions.socketsMessageReceiving(msg));
  };
```
 
в самом мидлваре пишем 

```js
      case 'SOCKETS_MESSAGE_SEND':
        console.log('action');
        console.log(action);
        socketExample.send(action.message_send);
        store.dispatch(SocketExampleActions.socketsMessageSending(action.message_send));
        break;
```

Добавляем обработчик для нашего сокета

```js
      case 'SOCKETS_CONNECT':
        if (socketExample !== null) {
          console.log('SOCKETS_DISCONNECTING');
          store.dispatch(SocketExampleActions.socketsDisconnecting());
          socket.close();
        }
        console.log('SOCKETS_CONNECTING');
        socketExample = new WebSocket('wss://echo.websocket.org/');
        store.dispatch(SocketExampleActions.socketsConnecting());
        socketExample.onmessage = onMessage(socketExample, store);
        socketExample.onclose = onClose(store);
        socketExample.onopen = onOpen(action.token);
        break;
```

Окей, становится непонятно. `action.message_send` - это о чем? так случилось, что все что мы кладем в редукс появляется в процессе обработки `store => next => action =>` в этих переменных.

Давайте разберемся как в action появится сообщение.

Правим аналогично подключению файл `src\components\SocketExampleComponents\SocketMessageLog.js`

```js
  static propTypes = {
    loaded: PropTypes.bool,
    message: PropTypes.string,
    connected: PropTypes.bool,
    message_history: PropTypes.array,
    socketsMessageSend: PropTypes.func,
  }
```
да нем не нужно сообщение о получение, мы будем его запускать из мидлваре

```js
  handleSendButton = (event) => {
    event.preventDefault();
    this.props.socketsMessageSend(this.refs.message_text.value);
    this.refs.message_text.value = '';
  }
```
аналогично мы получим новые сообщения сразу их редукса. Здесь мы вызываем `this.props.socketsMessageSend(this.refs.message_text.value)` тем самым в `action` мы передаем наше сообщение.

тестим - должно все получиться. 
Коммитимся.

Финиш!
Оглянитесь на себя в начале этой статьи - надеюсь, что у вас появилось много интересных и гениальных задумок, не откладывайте - делайте. Я вот эту статью пишу и публикую в первый раз и крайне переживаю и откровенно нервничаю, как ее встретя. А вдруг я это все не зря делал ;)










