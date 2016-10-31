С чатом мы повторим все теже упражнения, что подключением добавим историю и наичим ее выводить. 

Полный цикл будет выглядеть так: 
* В редюсере нужно будет определить новую переменную в редюсере, 
* описать для нее экшены,
* описать как данная переменна будет изменяться. 
* В компоненте нужно будет подхватить эту переменную
* включить ее в отображение 
* связать кнопку на интерфейсе и экшены в редуксе

Начнем с файле `src\redux\modules\socketexamplemodule.js` нам нужно

добавить новую переменную 
```js
const initialState = {
  loaded: false,
  message: 'Just created',
  connected: false,
  history: [],
  message_history: []
};
```
так как у нас уже есть экшены `SOCKETS_MESSAGE_SENDING` и  `SOCKETS_MESSAGE_RECEIVING`, то теперь нам нужно просто описать как будет работать редюсер.
```js
    case SOCKETS_MESSAGE_SENDING:
      return Object.assign({}, state, {
        loaded: true,
        message: 'Send message',
        connected: true,
        message_history: [
          ...state.message_history,
          {
            direction: '->',
            message: action.message_send
          }
        ]
      });
    case SOCKETS_MESSAGE_RECEIVING:
      return Object.assign({}, state, {
        loaded: true,
        message: 'Message receive',
        connected: true,
        message_history: [
          ...state.message_history,
          {
            direction: '<-',
            message: action.message_receive
          }
        ]
      });
  ```
Мы обращаем внимание на странные переменный `action.message_receive` и `action.message_send`, с помощью которых мы изменяем состояние. Эти переменные мы получаем из экшенов следующим путем
```js
export function socketsMessageSending(sendMessage) {
  return {type: SOCKETS_MESSAGE_SENDING, message_send: sendMessage};
}
export function socketsMessageReceiving(sendMessage) {
  return {type: SOCKETS_MESSAGE_RECEIVING, message_receive: sendMessage};
}
```
Т.е. откуда-то из кода мы будем запускать эти экшены и передавать им по одной переменной `sendMessage` или `sendMessage`.


Как и договорились будем запускать экшены по нажатию кнопок. Пока мы просто эмулируем работу чата на стороне клиента и постепенно у нас получается прототип будущего приложения.

На этом коммитимся пока.

Продолжаем уже теперь будем связывать это все с нашим элементом `SocketMessageLog.js`, который взаимодейсвует напрямую с редюксом, поэтому мы все реализуем в одном файле. 

Подключаем новую переменную, которую мы получаем из редюкса
```js
@connect(
  state => ({
    loaded: state.socketexample.loaded,
    message: state.socketexample.message,
    connected: state.socketexample.connected,
    message_history: state.socketexample.message_history
  }),
  socketExampleActions)
export default class SocketMessageLog extends Component {
  static propTypes = {
    loaded: PropTypes.bool,
    message: PropTypes.string,
    connected: PropTypes.bool,
    message_history: PropTypes.array
  }
```

Теперь нам нужны функции, которые будут обрабатывать нажатия кнопок на форме
```js
  handleSendButton = (event) => {
    event.preventDefault();
    this.props.socketsMessageSending(this.refs.message_text.value);
    this.refs.message_text.value = '';
  }
```
Забираем по ссылке `message_text` и тут же стираем ее значние для ввода нового. Главное, что мы передаем `message_text` в наш экшен оправки сообщения. 
Прописываем рендер и тестируем. 
Не забываем
```js
const {loaded, message, connected, message_history} = this.props;
```
Выводим лог сообщений, по аналогии с подключением
```js
        <ul>
          {
            message_history.map((messageHistoryElement, index) =>
            <li key={index} className={'unstyled'}>
              <span className={(messageHistoryElement.direction === '->') ? 'glyphicon glyphicon-arrow-right' : 'glyphicon glyphicon-arrow-left'}></span>
            {messageHistoryElement.message}</li>
          )}
        </ul>
```

> Не пытайтесь использовать более вложенные ветвления - это у вас не получится. Вас будут от этого защищать по причине того, что здесь не место вычислений данных. Здесь вообще про интерфейс.

Обновляем форму и кнопки
```js
        <form
          className="form-inline"
          onSubmit={this.handleSendButton}>
          <p></p>
          <div className="form-group">
            <input
              className="form-control input-sm"
              type="text"
              ref="message_text" readOnly = {(loaded && connected === true) ? false : true}>
            </input>
          </div>
          <button
            className="btn btn-primary btn-sm"
            onClick={this.handleSendButton}
            disabled = {(connected === true) ? false : true}>
            <i className="fa fa-sign-in"/> Send
          </button>
        </form>
```
Тестируем и видим отправленные сообщения. Давайте сэмулируем пока получение сообщение. Будем делать это в лоб.
```js
  handleSendButton = (event) => {
    event.preventDefault();
    this.props.socketsMessageSending(this.refs.message_text.value);
    this.props.socketsMessageReceiving(this.refs.message_text.value);
    this.refs.message_text.value = '';
  }
```
и не забываем добавить новые элементы в проверку!
```js
  static propTypes = {
    loaded: PropTypes.bool,
    message: PropTypes.string,
    connected: PropTypes.bool,
    message_history: PropTypes.array,
    socketsMessageSending: PropTypes.func,
    socketsMessageReceiving: PropTypes.func
  }
```

Коммитимся. 

Отлично, скучная часть закончилась!