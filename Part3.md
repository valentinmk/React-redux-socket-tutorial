# Часть третья. Изумительный Redux

Переходим сразу к Redux

Для этого нужно: 
1. Cоздать редусер   
1. Создать экшены  
1. И подключить все это в общую систему


> Про экшены написано [В официальной документации](http://redux.js.org/docs/basics/Actions.html)  
> Про редюсеры написано [Там же](http://redux.js.org/docs/basics/Reducers.html)

### Создаем файл

Создаем файл `./src/redux/modules/socketexamplemodule.js` и наполняем базовыми экшенами и редюсерами. Вот тут базовом примере есть странная нестыковка, все предлагается писать в одном файле не разделяя на файл экшенов и редюсеров, ну допустим. Все равно - мы тут все взрослые люди (we are all adults).

### Экшены 1

```js
export const SOCKETS_CONNECTING = 'SOCKETS_CONNECTING';
export const SOCKETS_DISCONNECTING = 'SOCKETS_DISCONNECTING';
export const SOCKETS_MESSAGE_SENDING = 'SOCKETS_MESSAGE_SENDING';
export const SOCKETS_MESSAGE_RECEIVING = 'SOCKETS_MESSAGE_RECEIVING';
```

Все эти экшены мы будем запускать по нажатию кнопок и `SOCKETS_MESSAGE_RECEIVING мы синтетически вызывать вслед за отпракой сообщения.`

### Редюсер

```js
export default function reducer(state = initialState, action = {}) {
  switch (action.type) {
    case SOCKETS_CONNECTING:
      return Object.assign({}, state, {
        loaded: true,
        message: 'Connecting...',
        connected: false
      });
    case SOCKETS_DISCONNECTING:
      return Object.assign({}, state, {
        loaded: true,
        message: 'Disconnecting...',
        connected: true
      });
    case SOCKETS_MESSAGE_SENDING:
      return Object.assign({}, state, {
        loaded: true,
        message: 'Send message',
        connected: true
      });
    case SOCKETS_MESSAGE_RECEIVING:
      return Object.assign({}, state, {
        loaded: true,
        message: 'Message receive',
        connected: true
      });
    default:
      return state;
  }
}
```

Более подробно про структуру reducer можно прочитать тут. Более подробно зачем `Object.assign({}, state,{}); можно прочитать тут.`

Вы заметили инициализацию state = initialState добавим ее до объявления редюсера. Это будет первое состояние, которое мы будем иметь в нашем redux на момент загрузки страницы, ну точнее она будет с этим значение загружаться.

```js
const initialState = {
  loaded: false,
  message: 'Just created',
  connected: false,
};
```

### Экшены 2

Ну а теперь продолжим с нашими экшенами и на этом завершим этот модуль.

```js
export function socketsConnecting() {
  return {type: SOCKETS_CONNECTING};
}
export function socketsDisconnecting() {
  return {type: SOCKETS_DISCONNECTING};
}
export function socketsMessageSending() {
  return {type: SOCKETS_MESSAGE_SENDING};
}
export function socketsMessageReceiving() {
  return {type: SOCKETS_MESSAGE_RECEIVING};
}
```

### Подключаем в общий редюсер

На данный момент в приложении ничего не поменяется. Включаем наш модуль в общий reducer.

В фале `src\redux\modules\reducer.jsпрописываем модуль`

```js
import socketexample from './socketexamplemodule';
```

и включаем его в общую структуру результатирующего редюсера

```js
export default combineReducers({
  routing: routerReducer,
  reduxAsyncConnect,
  auth,
  form,
  multireducer: multireducer({
    counter1: counter,
    counter2: counter,
    counter3: counter
  }),
  info,
  widgets,
// our hero
  socketexample
});
```

Запускаем, проверяем и ура в DevTools мы видим.

![](redux_new.tiff)

Если вопросы с initialState остались попробуйте их поменять или добавить новую переменную в него.

Коммитимся на этом моменте.

### Стор

А стор у нас уже создан и редюсер в него подключен. Ничего не делаем.

## Подключаем redux к react компонентам

Подключаем все это теперь в наши модули. В целях всесторонней демонстрации начнем с модуля истории и сделаем из него чистый компонент react \(в смысле чисты от redux\).

Компонент `SocketConnectionLog мы пока не трогаем, а идем сразу в контейнер SocketExamplePage.`

В данном контейнере мы будем подключать и получать данные из redux.

Подключаем библиотеку

```js
import {connect} from 'react-redux';
```

Забираем экшены, чтобы потом их использовать у себя в react

```js
import * as socketExampleActions from 'redux/modules/socketexamplemodule';
```

а еще мы поменяем строку, чтобы подключить PropTypes

```js
import React, {Component, PropTypes} from 'react';
```

Пишем конектор, которым будем все забирать

```js
@connect(
  state => ({
    loaded: state.socketexample.loaded,
    message: state.socketexample.message,
    connected: state.socketexample.connected}),
  socketExampleActions)
```

Как вы видите state.socketexample.loaded это обращение в redux, в той струтуре которую мы видим в DevTools.

Теперь подключаем проверки получаемых из redux данных, что видится целесообразным т.к. любые проверки данных на тип есть гуд.

```js
static propTypes = {
   loaded: PropTypes.bool,
   message: PropTypes.string,
   connected: PropTypes.bool
 }
```

Мы получили данные теперь давайте их передавать. Внутри блока render объявляем и принимаем данные уже теперь из props

```js
const {loaded, message, connected} = this.props;
```

и спокойно и уверенно передаем их в наш модуль:

```js
<SocketConnectionLog loaded={loaded} message={message} connected={connected} />
```

Теперь переписываем наш компонент, который уже ничего не знает про redux

В файле `src\components\SocketExampleComponents\SocketConnectionLog.jsдействуевуем по списку:`

1. проверяем полученые props 
2. присваиваем их внутри render 
3. используем в нашем компоненте

Импортируем недостающие библиотеки:

```js
import React, {Component, PropTypes} from 'react';
```

добавляем проверку:

```js
static propTypes = { 
    loaded: PropTypes.bool,  
    message: PropTypes.string,  
    connected: PropTypes.bool  
}
```

объявляем и присваиваем, переданные через props переменные

```js
const {loaded, message, connected} = this.props;
```

используем для вывода наши переменные

```js
value={'index =' + 0 + ', loaded = ' + loaded + ', message = ' + message + ', connected = ' + connected}/>
          {/* value="
            index = 2, loaded = true, message = Connected, connected = true
            index = 1, loaded = false, message = Connecting..., connected = false"/>
          */}
```

Проверяем и видим, initialState прилетает к нам прямо из redux-&gt;react-&gt;props-&gt;props

Коммитимся на этом моменте.

### SocketExampleMessageLog

Теперь переходимо к компоненту `SocketExampleMessageLogи сделаем его абсолютно самостоятельным относительно react. Мы не будем передавать в него никакие props, он будет получать все, что ему нужно через redux.`

Открываем файл src\components\SocketExampleComponents\SocketMessageLog.js

в нем добавляем необходимые нам библиотеки

```js
import React, {Component, PropTypes} from 'react';
import * as socketExampleActions from 'redux/modules/socketexamplemodule';
```

добавляем `connect, проверку типов и испольщуем полученные данные`

```js
@connect(
 state => ({
 loaded: state.socketexample.loaded,
 message: state.socketexample.message,
 connected: state.socketexample.connected}),
 socketExampleActions)
export default class SocketMessageLog extends Component {
  static propTypes = {
    loaded: PropTypes.bool,
    message: PropTypes.string,
    connected: PropTypes.bool
  }
  render() {
    const {loaded, message, connected} = this.props;
 // ...
```

Мы будем использовать `loaded  и connected, чтобы определять готовность к обмену сообщения, а message выведем просто для проверки.`

```auto
<ul>
 <li key="1" className="unstyled">
  <span className="glyphicon glyphicon-arrow-right"> </span>
   {message}
 </li>
 <li key="2" className="unstyled">
  <span className="glyphicon glyphicon-arrow-left"> </span>
   [ECHO] {message}
  </li>
 </ul>
```

Я буду проверять переменные loaded и connected явно, чтобы быть более прозрачным для потомков.

```js
<form className="form-inline">
 <p></p>
 <div className="form-group">
  <input
   className="form-control input-sm"
   type="text"
   ref="message_text" readOnly = {(loaded === true) ? false : true}></input>
 </div>
 <button
  className="btn btn-primary btn-sm"
  disabled = {(connected === true) ? false : true}>
  <i className="fa fa-sign-in"/> Send
 </button>
</form>
```

Полпути пройдено. Коммитимся.

