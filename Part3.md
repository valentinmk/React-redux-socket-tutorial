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

Все экшены мы будем запускать по нажатию кнопок, кроме события `SOCKETS_MESSAGE_RECEIVING`, который мы будем синтетически вызывать вслед за отпракой сообщения. Это делается, чтобы в процессе разработки эмулировать недостающие в настоящий момент (или на конкретном этапе) функционал серверной части приложения.

### Редюсер

Добавляем в тот же файл.
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

Более подробно про структуру reducer и зачем `Object.assign({}, state,{});` можно прочитать [тут](http://redux.js.org/docs/basics/Reducers.html).

Вы заметили инициализацию state = initialState, которой мы не объявили (поставьте ESLint или его аналог - сильно упростит жизнь Нормального Человека). Добавим объявление до редюсера. Это будет первое состояние, которое мы будем иметь в нашем redux на момент загрузки страницы, ну точнее страница будет загружаться уже с этим первоначальным состоянием.

```js
const initialState = {
  loaded: false,
  message: 'Just created',
  connected: false,
};
```

### Экшены 2

Теперь продолжим с нашими экшенами и на этом завершим этот модуль. Мы должны описать, как они будут изменять состояние reducer'a.

Добавляем в тот же файл.
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

На данный момент в приложении ничего не поменяется. Включаем наш модуль в общий контструктор reducer'ов.

В фале `./src/redux/modules/reducer.js` прописываем модуль.

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
  pagination,
  widgets,  
// our hero
  socketexample
});
```

Запускаем сервер, проверяем и ура в DevTools мы видим.

![](redux_new.tiff)

Если вопросы с initialState остались, то попробуйте их поменять или добавить новую переменную в него.

Коммит: [https://github.com/valentinmk/react-redux-universal-hot-example/commit/0c984e3b5bc25056aa578ee57f90895bc6baaf18](https://github.com/valentinmk/react-redux-universal-hot-example/commit/0c984e3b5bc25056aa578ee57f90895bc6baaf18)

### Стор

А стор у нас уже создан и редюсер в него подключен. Ничего не делаем.
 
Если подробнее, то вы должны помнить как мы добавили наш редюсер в `combineReducers` выше по статье. Так вот этот `combineReducers` сам включается в стор, который создаётся в файле `./src/redux/create.js`.


## Подключаем redux к react компонентам

Подключаем все это теперь в наши модули. В целях всесторонней демонстрации начнем с модуля истории и сделаем из него чистый компонент react (в смысле чистый от redux).

Компонент `SocketConnectionLog` мы пока не трогаем, а идем сразу в контейнер `SocketExamplePage`.

В данном контейнере мы будем подключать и получать данные из redux.

Подключаем библиотеку в файле `./src/containers/SocketExample/SocketExamplePage.js`.

```js
import {connect} from 'react-redux';
```

Забираем экшены, чтобы потом их использовать у себя в react.

```js
import * as socketExampleActions from 'redux/modules/socketexamplemodule';
```

а еще мы поменяем строку, чтобы подключить PropTypes

```js
import React, {Component, PropTypes} from 'react';
```

Пишем конектор, которым будем забирать данные из нашего редюсера.

```js
@connect(
  state => ({
    loaded: state.socketexample.loaded,
    message: state.socketexample.message,
    connected: state.socketexample.connected}),
  socketExampleActions)
```

Как вы видите `state.socketexample.loaded` это обращение в redux, в той структуре, которую мы видим в DevTools.

Теперь подключаем проверки данных, получаемых из redux, что видится целесообразным т.к. любые проверки данных на тип есть вселенское добро.

```js
  static propTypes = {
    loaded: PropTypes.bool,
    message: PropTypes.string,
    connected: PropTypes.bool
  }
```

Мы получили данные теперь давайте их передавать. Внутри блока render объявляем и принимаем данные уже теперь из props.

```js
const {loaded, message, connected} = this.props;
```

и спокойно и уверенно передаем их в наш модуль:

```js
<SocketConnectionLog loaded={loaded} message={message} connected={connected} />
```

Мы передали новые данные (через react) в компонент. Теперь переписываем наш компонент, который уже ничего не знает про redux, а только обрабатывает переданные ему данные.

В файле `./src/components/SocketExampleComponents/SocketConnectionLog.js` действуем по списку:

1. проверяем полученые props 
2. присваиваем их внутри render 
3. используем в нашем компоненте

Начнем, импортируем недостающие библиотеки:

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

объявляем и присваиваем переменные, переданные через props

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

Проверяем и видим, initialState прилетает к нам прямо из redux->react->props->props. 

Коммит: [https://github.com/valentinmk/react-redux-universal-hot-example/commit/60ac05332e35dfdbc11b9415f5bf5c46cd740ba8](https://github.com/valentinmk/react-redux-universal-hot-example/commit/60ac05332e35dfdbc11b9415f5bf5c46cd740ba8).

### SocketExampleMessageLog

Теперь переходим к компоненту `SocketExampleMessageLog` и сделаем его абсолютно самостоятельным, в смысле работы с redux. Мы не будем передавать в него никакие props, он будет получать все, что ему нужно через redux сам.

Открываем файл `./src/components/SocketExampleComponents/SocketMessageLog.js`

в нем добавляем необходимые нам библиотеки

```js
import React, {Component, PropTypes} from 'react';
import {connect} from 'react-redux';
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
 // ...
```
Не забываем передать значение в метод render() через props

```js
    const {loaded, message, connected} = this.props;
```

Мы будем использовать `loaded`  и `connected`, чтобы определять готовность к обмену сообщения, а `message` выведем просто для проверки.

```js
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

Я буду проверять переменные `loaded` и `connected` явно, чтобы быть более прозрачным для (возможных) потомков.

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

Полпути пройдено. 
Коммит: [https://github.com/valentinmk/react-redux-universal-hot-example/commit/a473d6a86262f2d2b52c590974e77df9454de5a1](https://github.com/valentinmk/react-redux-universal-hot-example/commit/a473d6a86262f2d2b52c590974e77df9454de5a1).

