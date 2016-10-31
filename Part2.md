# Часть вторая. Проектирование будущего приложения


Даня Абрамов писал много про то, что и как нужно разделять в приложении и как организовывать структуру приложения.

В прекрасном руководстве для начинающих React представлен подход по проектированию динамических приложений на React, от которого мы не будем отклоняться, а прямо будем следовать по нему.


###Итак начнем.

Прежде всего хочу сказать, что я приверженец наглядности и отладки прямо при написании приложения, поэтому некоторое количество компонентов мы уберем после окончания работы убрать с формы.

###Зачем? (определяем назначение, ставим цель, обозначаем задачи, делаем предоложения и упрощения)

Нам очень сильно интересно Как можно реализовать взаимодействие с Web socket с использованием React в связке с Redux.

Наше приложение должно подключаться к серверу реализующему WebSocket и производить обмен сообщениями.

Упрощения и допущения:
* Мы не будем использовать авторизации в приложении
* Мы не будет использовать авторизации в WebSocket-ах 
* Мы будем использовать самое доступное приложение Websocket Echo ([https://www.websocket.org/echo.html](https://www.websocket.org/echo.html))

### Пользовательский интерфейс 1

Мы добавляем два новых раздела на нашу страницу. 

* В логе подключения сокетов будем кратко выводить текущие события связанные с подключением отключением. 

```js
// src/components/SocketExamplePage.js
// inside render () { return (...) }
        <h3>Socket connection log</h3>
          <textarea className="form-control" rows="1" readOnly placeholder="Waiting ..."
            value="
            index = 2, loaded = true, message = Connected, connected = true
            index = 1, loaded = false, message = Connecting..., connected = false"/>
```

> index - порядковый номер записи лога
> loaded - загружен ли элемент на странице пользователя
> message - переменна для отладки и наглядности кода
> connected - признак подключены ли мы сейчас к серверу 


* В логе сообщений будем отображать отправленные `->` и полученные сообщения `<-`.

```js
// src/components/SocketExamplePage.js
// inside render () { return (...) }
      <h3>Message log</h3>
        <ul>
            <li key="1" className="unstyled">
              <span className="glyphicon glyphicon-arrow-right"></span>
              Socket string
            </li>
            <li key="2" className="unstyled">
              <span className="glyphicon glyphicon-arrow-left"></span>
              [ECHO] Socket string
            </li>
        </ul>
```

### Пользовательский интерфейс 2

Конечно мы забыли про кнопки и поля ввода, добавляем:
* подключиться к websocket
* отключиться от websocket

и немного наведем порядка
```js
      <h3>Socket connection log</h3>
          <textarea
          className="form-control"
          rows="1"
          readOnly
          placeholder="Waiting ..."
          value="
            index = 2, loaded = true, message = Connected, connected = true
            index = 1, loaded = false, message = Connecting..., connected = false"/>
          <button className="btn btn-primary btn-sm">
            <i className="fa fa-sign-in"/> Connect
          </button>
          <button className="btn btn-danger btn-sm">
            <i className="fa fa-sign-out"/> Disconnect
          </button>
```


* отправить сообщение

```js
        <h3>Message log</h3>
        <ul>
            <li key="1" className="unstyled">
              <span className="glyphicon glyphicon-arrow-right"></span>
              Socket string
            </li>
            <li key="2" className="unstyled">
              <span className="glyphicon glyphicon-arrow-left"></span>
              [ECHO] Socket string
            </li>
        </ul>
        <form className="form-inline">
          <p></p>
          <div className="form-group">
            <input
            className="form-control input-sm"
            type="text"
            ref="message_text"></input>
          </div>
          <button className="btn btn-primary btn-sm">
            <i className="fa fa-sign-in"/> Send
          </button>
        </form>
```

> Не нажимайте кнопку Send

Окей, закомитимся для получения полного кода.

### Компоненты

Давайте разделим все на компоненты. Ничего сложного.

Создаем новую папку в директории `src\components` назовем ее `SocketExampleComponents`. 
Добавление компонента происходит в три шага
1. создаем файл с компонентом в нашей папке `SocketConnectionLog.js`

> мы оборачиваем в div так как от нас этого ожидает React

```js
import React, {Component} from 'react';

export default class SocketConnectionLog extends Component {
  render() {
    return (
      <div>
        <h3>Socket connection log</h3>
          <textarea
          className="form-control"
          rows="1"
          readOnly
          placeholder="Waiting ..."
          value="
            index = 2, loaded = true, message = Connected, connected = true
            index = 1, loaded = false, message = Connecting..., connected = false"/>
          <button className="btn btn-primary btn-sm">
            <i className="fa fa-sign-in"/> Connect
          </button>
          <button className="btn btn-danger btn-sm">
            <i className="fa fa-sign-out"/> Disconnect
          </button>
      </div>
    );
  }
}
```

2. ВАЖНО. Прописываем наш новый компонент в файле `components\index.js`

```js
export SocketConnectionLog from './SocketExampleComponents/SocketConnectionLog';
```

3. Правим нашу страницу и вместо скопированного нами кода вставляем только один элемент
```js
// src/components/SocketExamplePage.js
// ...
import {SocketConnectionLog} from 'components';
// ...
      <SocketConnectionLog />
```

Добавляем другой новый компонент в ту же папку `src\components\SocketExampleComponents`. 
Добавляем в три шага
1. Cоздаем файл с компонентом в нашей папке `SocketMessageLog.js`

```js
import React, {Component} from 'react';

export default class SocketMessageLog extends Component {
  render() {
    return (
      <div>
        <h3>Message log</h3>
        <ul>
            <li key="1" className="unstyled">
              <span className="glyphicon glyphicon-arrow-right"></span>
              Socket string
            </li>
            <li key="2" className="unstyled">
              <span className="glyphicon glyphicon-arrow-left"></span>
              [ECHO] Socket string
            </li>
        </ul>
        <form className="form-inline">
          <p></p>
          <div className="form-group">
            <input
            className="form-control input-sm"
            type="text"
            ref="message_text"></input>
          </div>
          <button className="btn btn-primary btn-sm">
            <i className="fa fa-sign-in"/> Send
          </button>
        </form>
      </div>
    );
  }
}
```
2. Прописываем наш новый компонент в файле `components\index.js`

```js
export SocketMessageLog from './SocketExampleComponents/SocketMessageLog';
```

3.Правим нашу страницу и вместо скопированного нами кода вставляем только один элемент

```js
// src/components/SocketExamplePage.js
// ...
import {SocketMessageLog} from 'components';
// ...
        <SocketMessageLog/>
```

Проверяем. Ничего не изменилось - это успех.



Коммитимся и заканчиваем 2 часть.





