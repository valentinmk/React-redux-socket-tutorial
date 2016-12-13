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
Подробнее. Вообще мидлваре управляет самим редуксом и как он обрабатывает события и состояния внутри себя. Использую конструкцию `return store => next => action =>` мы вмешиваемся в каждый экшен происходящий в редуксе и по полю `switch (action.type)` выполняем те или иные действия.

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

Запускаем проверяем. Теперь в консоле полный беспорядок и это означает, что наш концепт работает!
Коммит: [https://github.com/valentinmk/react-redux-universal-hot-example/commit/7833a405be3445e58e8e672e9db03f8cfbfde022](https://github.com/valentinmk/react-redux-universal-hot-example/commit/7833a405be3445e58e8e672e9db03f8cfbfde022)

### Второй проход. Делаем лог историю.

Мы проверили концепцию и готовы делать нашу боевую модель. Теперь будемподключаться к websockets. 


> Здесь и далее даются вырианты написания кода, которые иллюстрируют ход разработки. Эти примеры содержат преднамеренные ошибки, которые показывают основные технические проблемы и особенности, с которыми я столкнулся в рамках подготовительных работ.


Добавляем в файл `./src/redux/middleware/socketExampleMiddleware.js` функции, которыми будем обрабатывать события подключения и отключения.

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

убираем лишние объявления (нужно удалить)
```js
  socketExample = true;
  socketExampleActions();
```

добавляем наши редюсеры и убираем лишнее логирование.

```js
      case 'SOCKETS_CONNECT':
        if (socketExample !== null) {
          console.log('SOCKETS_DISCONNECTING');
          store.dispatch(socketExampleActions.socketsDisconnecting());
          socket.close();
        }
        console.log('SOCKETS_CONNECTING');
        socketExample = new WebSocket('ws://echo.websocket.org/');
        store.dispatch(socketExampleActions.socketsConnecting());
        socketExample.onclose = onClose();
        socketExample.onopen = onOpen(action.token);
        break;
      default:
        return next(action);
```

Подробнее. Начинаем разбираться. Мы ловим событие `SOCKETS_CONNECT`, проверяем подключеныли мы, если нет то запускаем принудительное закрытие подключения, создаем новый вебсокет и добавляем ему методы `onClose()` и `onOpen(action.token)`. Понимает, что сейчас ничего не работает. Мы ловим экшен `SOCKETS_CONNECT`, которого у нас пока нет. Но у нас есть другой экшен `SOCKETS_CONNECTING`, почему бы не использовать его - меняем скрипт. 

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


> !!! Внимание после этого скрипт будет находиться в бесконечном цикле - сохраните все или не нажимайте кнопку подключиться на этом этапе.

Проверяем и видим, что все пошло не так. В консоле постоянные `SOCKETS_CONNECTING` и `SOCKETS_DISCONNECTING`. Закрываем вкладку или браузер. 
Подробнее. Мидлваре "слушает" стор на предмет экшенов `store => next => action =>` и включается в обработку, когда находит свой экшен `SOCKETS_CONNECTING`. Далее по коду идет вызов экшена `store.dispatch(SocketExampleActions.socketsConnecting());`, который в свою очередь вызывает экшен `SOCKETS_CONNECTING`, который ловит мидлваре и т.д.  


Вывод простой - экшены для мидлеваре должны быть всегда отдельными от экшенов, которые происходят на стороне клиентов.


Как быть дальше.

Наш вариант (я думаю он не один) будет таким: 
* пользователь будет вызывать нажатием кнопки экшены мидлвара,
* который будет вызывать уже "интерфейсные" экшены. 

Что на практике будет означать 
* `SOCKETS_CONNECT` вызывается пользователем
* при его обработки будет вызываться `SOCKETS_CONNECTING`,
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

Теперь нужно дать возможность пользователю запускать данные действия. По идеи нужно лезть в `./src/components/SocketExampleComponents/SocketConnectionLog.jsр`, но на самом деле управляющие функции ему передают через компонент react. Поэтому правим сначала `./src/containers/SocketExample/SocketExamplePage.js`. 

```js
static propTypes = {
    loaded: PropTypes.bool,
    message: PropTypes.string,
    connected: PropTypes.bool,
    history: PropTypes.array,
    socketsConnecting: PropTypes.func,
    socketsDisconnecting: PropTypes.func,
//HERE
    socketsConnect: PropTypes.func,
    socketsDisconnect: PropTypes.func
  }
  render() {
//HERE
    const {loaded, message, connected, socketsConnecting, socketsDisconnecting, history, socketsConnect, socketsDisconnect} = this.props;
    return (
      <div className="container">
        <h1>Socket Exapmle Page</h1>
        <Helmet title="Socket Exapmle Page"/>
        <SocketConnectionLog
          loaded={loaded}
          message={message}
          connected={connected}
          connectAction={socketsConnecting}
          disconnectAction={socketsDisconnecting}
          history={history}
//HERE
          connectAction={socketsConnect}
          disconnectAction={socketsDisconnect}
          />
        <SocketMessageLog/>
      </div>
    );
  }
```

Возвращаемся к `./src/redux/middleware/SocketExampleMiddleware.js` и наводим порядок. 

Изменяем один кейс
```js
      case 'SOCKETS_CONNECT':
```
Добавляем кейс на обработку отключения:

```js
      case 'SOCKETS_DISCONNECT':
        if (socketExample !== null) {
          console.log('SOCKETS_DISCONNECTING');
          store.dispatch(socketExampleActions.socketsDisconnecting());
          socketExample.close();
        }
        socketExample = null;
        break;
```

Для того, чтобы иметь возможность запускать пользовательские экшены при событии disconnect передаем в системное событие также сам стор.

```js
        socketExample.onclose = onClose(store);
```

и изменяем сам обработчик
```js
  const onClose = (store) => evt => {
    console.log('WS is onClose');
    console.log('evt ' + evt.data);
    store.dispatch(socketExampleActions.socketsDisconnect());
  };
```
Ну вроде все - проверяем. Нужно использовать закладку Network или ее аналог в вашем браузере, чтобы увидеть подключения к вебсокетам. 

Для тестирования давайте проверим, что будет если мы на самом деле не смогли подключиться к сокетам. 
```js
        socketExample = new WebSocket('ws://echo.websocket.org123/');
```
Подробнее. Эта проверка связана с тем, что обработка событий у нас идет в асинхронном режиме. Мы не знаем в каком порядке от сокета нам будут прилетать события - последовательно, в обратном порядке или парами. Наш код должен быть способным корректно обрабатывать любые варианты. Попродуйте самостоятельно переместить `store.dispatch(socketExampleActions.socketsDisconnect());` из метода `onClose` в кейс редюсера и посмотреть что же изменится.

Коммит: [https://github.com/valentinmk/react-redux-universal-hot-example/commit/7569536048df83f7e720b000243ed9798308df20](https://github.com/valentinmk/react-redux-universal-hot-example/commit/7569536048df83f7e720b000243ed9798308df20)

### Проход второй. Делаем сообщения

Все аналогично первой части второго прохода.
Добавляем экшены в `./src/redux/modules/socketexamplemodule.js`

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
Стоп. Почему не 4 обработчика? Подробнее. Нам, на самом деле, нам не нужна обработка socketsMessageReceive, потому что пользователю не нужно вмешиваться в процесс получения сообщения сообщения. Хотя на будущее, если по-фантизировать этим событием мы можем отмечать факт отображения сообщения у пользователя в его интерфейсе, т.е. тот самый признак "прочитано" (но это за пределами этой статьи).

Переходим к описанию рпо в файле `src\redux\middleware\SocketExampleMiddleware.js` - все как обычно ничего необычного.
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










