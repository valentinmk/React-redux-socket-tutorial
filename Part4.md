# Часть четвертая. Оживляем историю

В предыдущих частя мы все подготовили к тому, чтобы начать использовать Redux.

В этой части мы будем связывать события в react и состояния в redux. Начнем.

Оживим историю подключений в нашем компоненте `./src/components/SocketExampleComponents/SocketConnectionLog.js`.
Но как мы помним, он ничего про redux не знает. Это означает, что он ничего не знает про экшены и поэтому ему их нужно передать через контейнер  `./src/containers/SocketExample/SocketExamplePage.js`. Просто передаем компоненту их как будто это простые props.

Вообще все функции экшенов мы подключили через @connect. Стоп. Подробней. Вспомним.
```js
//....
import * as socketExampleActions from 'redux/modules/socketexamplemodule';
//....
@connect(
  state => ({
    loaded: state.socketexample.loaded,
    message: state.socketexample.message,
    connected: state.socketexample.connected}),
  socketExampleActions)
```

Поэтому просто включаем их в проверку в файле `./src/containers/SocketExample/SocketExamplePage.js`: 

```js
static propTypes = {
  loaded: PropTypes.bool,
  message: PropTypes.string,
  connected: PropTypes.bool,
  socketsConnecting: PropTypes.func,
  socketsDisconnecting: PropTypes.func
 }
```
и передаем в наш компонент 

```js
  render() {
    const {loaded, message, connected, socketsConnecting, socketsDisconnecting} = this.props;
    return (
      <div className="container">
        <h1>Socket Exapmle Page</h1>
        <Helmet title="Socket Exapmle Page"/>
        <SocketConnectionLog loaded={loaded} message={message} connected={connected} connectAction={socketsConnecting} disconnectAction={socketsDisconnecting}/>
        <SocketMessageLog/>
      </div>
    );
  }
```

Теперь давайте обеспечим прием преданных в компонент экшенов в файле `./src/components/SocketExampleComponents/SocketConnectionLog.js`.
Мы будем добавлять их (экшены) в проверку и использовать в наших обработчиках действий на форме. Обработчиков сделаем два: по клику кнопки "Connect" и "Disconnect".

```js
  static propTypes = {
    loaded: PropTypes.bool,
    message: PropTypes.string,
    connected: PropTypes.bool,
    connectAction: PropTypes.func,
    disconnectAction: PropTypes.func
  }
  handleConnectButton = (event) => {
    event.preventDefault();
    this.props.connectAction();
  }
  handleDisconnectButton = (event) => {
    event.preventDefault();
    this.props.disconnectAction();
  }
```

Прописываем вызов обработчиков функций по нажатию соответствующих кнопок.

```js
  render() {
    const {loaded, message, connected} = this.props;
    return (
      <div>
        <h3>Socket connection log</h3>
        <textarea
          className="form-control"
          rows="1"
          readOnly
          placeholder="Waiting ..."
          value={'index =' + 0 + ', loaded = ' + loaded + ', message = ' + message + ', connected = ' + connected}/>
          {/* value="
            index = 2, loaded = true, message = Connected, connected = true
            index = 1, loaded = false, message = Connecting..., connected = false"/>
          */}
        <button
          className="btn btn-primary btn-sm"
          onClick={this.handleDisconnectButton}>
          <i className="fa fa-sign-in"/> Connect
        </button>
        <button
          className="btn btn-danger btn-sm"
          onClick={this.handleConnectButton}>
          <i className="fa fa-sign-out"/> Disconnect
        </button>
      </div>
    );
```

Запускаем. Проверяем. Ура, оно живо! Можно посмотреть в DevTools, что события создаются в redux. 

Если внимательно проследить как меняются состояния, то можно заметить, что компонент истории сообщений работает как-то не так (хотя он написан правильно). Дело в том, что при нажатии кнопки подключения у нас состояние connected = false, а при разрыве подключения у нас состояние connected = true. Давай-те поправим. 

Для этого в файле `./src/redux/modules/socketexamplemodule.js` правим странные строчки

```js
 case SOCKETS_CONNECTING:
  return Object.assign({}, state, {
   loaded: true,
   message: 'Connecting...',
   connected: true
  });
 case SOCKETS_DISCONNECTING:
  return Object.assign({}, state, {
   loaded: true,
   message: 'Disconnecting...',
   connected: false
  });
```
Ну теперь все работает правильно. 

>НО далее мы поменяем эти значения на исходные, это важный момент. Событие попытки подключения не тождественно состоянию подключено (да я кэп).

Реализуем историю подключения. Главное ограничение принцип работы самого redux. Мы нее можем изменять само состояние, но мы можем его целиком пересоздавать и присваивать. Поэтому чтобы накапливать историю мы будем ее копировать, прибавлять к копии текущее состояние и присваивать это значение оригиналу (с которого сняли копию).

```js
    case SOCKETS_CONNECTING:
      return Object.assign({}, state, {
        loaded: true,
        message: 'Connecting...',
        connected: true,
        history: [
          ...state.history,
          {
            loaded: true,
            message: 'Connecting...',
            connected: true
          }
        ]
      });
    case SOCKETS_DISCONNECTING:
      return Object.assign({}, state, {
        loaded: true,
        message: 'Disconnecting...',
        connected: false,
        history: [
          ...state.history,
          {
            loaded: true,
            message: 'Disconnecting...',
            connected: false
          }
        ]
      });
```

Делаем отображение в том же элементе. Прежде всего передаем переменную истории через props в файле `./src/containers/SocketExample/SocketExamplePage.js`. Далее в файле `./src/components/SocketExampleComponents/SocketConnectionLog.js` принимает переданную переменную.

Приступим в файле `./src/containers/SocketExample/SocketExamplePage.js` забираем из редусера:
```js
@connect(
 state => ({
  loaded: state.socketexample.loaded,
  message: state.socketexample.message,
  connected: state.socketexample.connected,
  history: state.socketexample.history
 }),
 socketExampleActions)
```
проверяем на тип 
```js
  static propTypes = {
    loaded: PropTypes.bool,
    message: PropTypes.string,
    connected: PropTypes.bool,
    history: PropTypes.array,
    socketsConnecting: PropTypes.func,
    socketsDisconnecting: PropTypes.func    
  }
```

присваиваем и передаем 

```js
  render() {
    const {loaded, message, connected, socketsConnecting, socketsDisconnecting, history} = this.props;
    return (
      <div className="container">
        <h1>Socket Exapmle Page</h1>
        <Helmet title="Socket Exapmle Page"/>
        <SocketConnectionLog loaded={loaded} message={message} connected={connected} connectAction={socketsConnecting} disconnectAction={socketsDisconnecting} history={history}/>
        <SocketMessageLog/>
      </div>
    );
```

Принимаем уже в файле `./src/components/SocketExampleComponents/SocketConnectionLog.js`.

```js
 static propTypes = {
   loaded: PropTypes.bool,
   message: PropTypes.string,
   connected: PropTypes.bool,
   history: PropTypes.array,
   connectAction: PropTypes.func,
   disconnectAction: PropTypes.func
 }
```
Для вывода истории в лог нам уже на самом деле не требуются текущие значения `loaded`, `message`, `connected`.

Давайте выведем в историю в обратной хронологии, так чтобы актуально состояние всегда было сверху. 

```js
  render() {
    const {history} = this.props;
    return (
      <div>
        <h3>Socket connection log</h3>
        <textarea
          className="form-control"
          rows="1"
          readOnly
          placeholder="Waiting ..."
          value={
            history.map((historyElement, index) =>
              'index = ' + index +
              ' loaded = ' + historyElement.loaded.toString() +
              ' message = ' + historyElement.message.toString() +
              ' connected = ' + historyElement.connected.toString() + ' \n').reverse()
            }/>
        <button
          className="btn btn-primary btn-sm"
          onClick={this.handleConnectButton}>
          <i className="fa fa-sign-in"/> Connect
        </button>
        <button
          className="btn btn-danger btn-sm"
          onClick={this.handleDisconnectButton}>
          <i className="fa fa-sign-out"/> Disconnect
          </button>
      </div>
    );
```
Главное, что нужно не забыть это добавить `history` при инициализации  редюсера, иначе наши проверки не будут срабатывать. 

В файле `./src/redux/modules/socketexamplemodule.js`.

```js
const initialState = {
  loaded: false,
  message: 'Just created',
  connected: false,
  history: []
};
```

Проверяем. И получаем нашу запись в истории подключения, но почему то с запятыми. Javascript, WTF? Ну да ладно, если мы добавим после мапа и реверса .join(''), то это все решает. 
> ".join('') все решает.", Карл!

Какой у нас результат? Читаем и пишем в redux! Можно себя похвалить! Но этого явно мало, ведь мы делаем это только внутри своей же собственной странички и никак не общаемся с внешним миром.

Коммит: [https://github.com/valentinmk/react-redux-universal-hot-example/commit/24144226ea4c08ec1af5db3a5e9b37461be2dbdd](https://github.com/valentinmk/react-redux-universal-hot-example/commit/24144226ea4c08ec1af5db3a5e9b37461be2dbdd)
