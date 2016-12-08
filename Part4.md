# Часть четвертая. Оживляем историю

В предыдущих частя мы все подгтовили к тому, чтобы начать использовать Redux в полной его силе.

В этой части мы будем связывать события в react и состояниями в redux. Начнем.

Оживим историю в нашем компоненте `./src/components/SocketExampleComponents/SocketConnectionLog.js`.
Но как мы помним он ничего про redux не знает и поэтому нужно передать ему экшены от редюсера через контейнер `src\containers\SocketExample\SocketExamplePage.js`.

Вообще все функции экшенов мы подключили через @connect, поэтому включаем их в проверку 
```js
static propTypes = {
  loaded: PropTypes.bool,
  message: PropTypes.string,
  connected: PropTypes.bool,
  socketsConnecting: PropTypes.func,
  socketsDisconnecting: PropTypes.func
 }
```
и передаем в наш комплонент 

```js
render() {
 const {loaded, message, connected, socketsConnecting, socketsDisconnecting} = this.props;
 return (
  <div className="container">
  <h1>Socket Example Page</h1>
  <Helmet title="Socket Exapmle Page"/>
  <p>Sockets not connected</p>
  <SocketConnectionLog loaded={loaded} message={message} connected={connected} connectAction={socketsConnecting} disconnectAction={socketsDisconnecting}/>
  <SocketMessageLog />
  </div>
 );
}
```

Теперь давайте свяжем все воедино уже в `src\components\SocketExampleComponents\SocketConnectionLog.js`
Мы будем добавлять в проверки и использовать в наших функция обработчика действий на форме.

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

мы будем вызывать наши функции по нажатию кнопок

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

Ура оно живо. Можно посмотреть в DevTools, что соответсвующие события создаются в redux. Если немного поиграться, то можно заметить, что компонент истории сообщений работает как-то не так (хотя он написан правильно). Давай-те поправим. 

Для этого в файле `src\redux\modules\socketexamplemodule.js` правим странные строчки

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
Ну теперь все работает правильно. НО далее мы поменяем эти значения на исходные, это важный момент, что событие подключения не тожственно состоянию подлючено (да я кэп).

Реализуем историю подключения. Мы можем накапливать информацию о в состоянии, но при этом мы не можем именять само состояние, хотя можем его целиком пересоздавать.

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

Делаем отображение тут же. В файле нашего `SocketConnectionLog.js`, а нет он же не знает про нашу историю, поэтому передаем как и прежде, в файле `SocketExamplePage.js`. Сам контейнер тоже не знает про историю ничего, поэтому:
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
присваиваем и передаем 

```js
// ...
const {loaded, message, connected, socketsConnecting, socketsDisconnecting, history} = this.props;
// ...
  <SocketConnectionLog loaded={loaded} message={message} connected={connected} connectAction={socketsConnecting} disconnectAction={socketsDisconnecting} history={history}/>
```

Принимаем в файле `SocketConnectionLog.js`
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
Давайте выведем в обратной хронологии. 

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
       }
     />
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

Проверяем. И получаем запятые, Javascript, WTF? Ну да ладно - .join('') все решает. 
> ".join('') все решает.", Карл!

Итого, читаем и пишем в redux, можно себя похвалить, но этого явно мало. Ведь мы делаем это только внутри себя.

Коиммитимся.