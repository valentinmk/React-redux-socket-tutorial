# Часть пятая. Проектируем чат 

У нас есть заготовка для подключения/отключения к сокету. Теперь мы должны сделать оболочку для чата, она станет нашим рабочем моделью (прототип у нас уже есть).

С чатом мы выполним такие же действия, как и с логом (историей) подключений - добавим историю чата и научим ее выводить.

Полный цикл будет выглядеть так: 
* В редюсере нужно: 
 * объявить новую переменную и инициализировать, 
 * описать для нее экшены,
 * описать как данная переменна будет изменяться. 
* В компоненте нужно:
 * принять эту переменную,
 * включить ее в отображение,
 * связать кнопку на интерфейсе и экшены в редуксе.

### Настройка редюсера

Начнем с файле `./src/redux/modules/socketexamplemodule.js` нам нужно
добавить новую переменную.
```js
const initialState = {
  loaded: false,
  message: 'Just created',
  connected: false,
  history: [],
  message_history: []
};
```
У нас уже есть экшены `SOCKETS_MESSAGE_SENDING` и  `SOCKETS_MESSAGE_RECEIVING`. Дополнительных экшенов создавать не будет. 

Приступаем к описанию, как будет себя вести нам нужно просто описать как будет работать редюсер.
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

Обратите внимание на переменные  переменный `action.message_receive` и `action.message_send`. С помощью них мы изменяем состояние нашего редюсера. Переменные будут передаваться внутри экшенов.

Реализуем передачу переменных в редюсер из экшенов.

```js
export function socketsMessageSending(sendMessage) {
  return {type: SOCKETS_MESSAGE_SENDING, message_send: sendMessage};
}
export function socketsMessageReceiving(sendMessage) {
  return {type: SOCKETS_MESSAGE_RECEIVING, message_receive: sendMessage};
}
```
Остановимся. Откуда-то из кода мы будем запускать эти экшены и передавать им по одной переменной `sendMessage` или `sendMessage`. Чтобы запустить эти экшены мы можем использовать абсолютно разные способы, но в нашем случае мы будем запускать экшены по нажатию кнопок. Пока мы просто моделируем работу чата на стороне клиента и постепенно у нас получается модель будущего приложения.

Мы выполнили работы со стороны редюсера и переходим к настройке отображения и управления из компонента.

### Настройка редюсера
Мы помним как для истории подключения мы использовали возможности react и передачу инфомрации из контейнера. В случае с сообщениями наш компонент сам по себе. 

Подключаем новую переменную, которую мы получаем из редюкса в файле `./src/components/SocketExampleComponents/SocketMessageLog.js`.
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
    message_history: PropTypes.array,
    socketsMessageSending: PropTypes.func
  }
```

Теперь нам нужны функции, которые будут обрабатывать нажатия кнопок на форме.
```js
  handleSendButton = (event) => {
    event.preventDefault();
    this.props.socketsMessageSending(this.refs.message_text.value);
    this.refs.message_text.value = '';
  }
```
Подробнее, забираем по ссылке из поля `message_text`. Передаем `message_text` в наш экшен оправки сообщения.  Стираем значние в этом поле для ввода нового.

Добавляем переменную в props.
```js
    const {loaded, connected, message_history} = this.props;
```
Выводим лог сообщений, по аналогии с подключением
```js
          <ul>
            {
              message_history.map((messageHistoryElement, index) =>
              <li key={index} className={'unstyled'}>
                <span className={(messageHistoryElement.direction === '->') ? 'glyphicon glyphicon-arrow-right' : 'glyphicon glyphicon-arrow-left'}></span>
                {messageHistoryElement.message}
              </li>
            )}
          </ul>
```

> Не пытайтесь использовать более вложенные ветвления - это у вас не получится. Т.е. не пытайтесь использовать вложенные ' '?' ':' '. Вас будут от этого защищать. Причина - здесь не место вычислений данных. Здесь вообще про интерфейс.

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
Тестируем и видим отправленные сообщения. 

Давайте сэмулируем пока получение сообщение. Будем делать это в лоб.
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