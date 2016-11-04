>Прежде чем начнем. Я все разрабатывал в обратном порядке - сначала крутил мидлваре, потом прокидывал экшены и только потом уже прикручивал адекватный интерфейс в reactjs. Мы в руководстве будем делать все в правильном порядке, потому что так действительно быстрее и проще. Минус моего подхода в том, что я использовал в разы больше отладки и "костылей", чем нужно на самом деле. Будем рациональными.

# Часть вторая. Проектирование будущего приложения

Сначала мы проектируем интерфейс пользователя. Для этого мы примерно представляем как должен выглядеть скелет нашего интерфейса и какие действия будут происходить в нем.

В руководстве для начинающих React представлен подход по проектированию динамических приложений на React, от которого мы не будем отклоняться, а прямо будем следовать по нему.

Дэн Абрамов писал в своей документации много про то, что и как нужно разделять в приложении и как организовывать структуру приложения. Мы будем следовать его примеру р

## Итак начнем.

Прежде всего хочу сказать, что для наглядности и отладки прямо при написании приложения мы будем добавлять элементы уберем с формы после окончания работы.


### Пользовательский интерфейс "Вариант 1"

Мы добавляем два новых раздела на нашу страницу.

В логе подключения сокетов будем кратко выводить текущие события связанные с подключением отключением. Изменяем файл `./src/containers/SocketExample/SocketExamplePage.js`.

```js
// inside render () { return (...) }
          <h3>Socket connection log</h3>
          <textarea
            className="form-control"
            rows="1"
            readOnly
            placeholder="Waiting ..."
            value="
              index = 2, loaded = true, message = Connected, connected = true
              index = 1, loaded = false, message = Connecting..., connected = false"/>
```

> index - порядковый номер записи лога

> loaded - признак загружен ли элемент на странице пользователя

> message - переменна-сообщение для отладки и наглядности кода

> connected - признак подключены ли мы сейчас к серверу

Конечно мы забыли про кнопки и поля ввода, добавляем:
* подключиться к websocket
* отключиться от websocket

```js
          <button className="btn btn-primary btn-sm">
            <i className="fa fa-sign-in"/> Connect
          </button>
          <button className="btn btn-danger btn-sm">
            <i className="fa fa-sign-out"/> Disconnect
          </button>
```

В логе сообщений будем отображать отправленные `->` и полученные сообщения `<-`.

```js
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

Кнопка и ввод для отправить сообщение

```js
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

Проверяем и закомитимся для получения полного кода.

Коммит [https://github.com/valentinmk/react-redux-universal-hot-example/commit/510a59f732a9bf42e070e7f57e970a2307661739](https://github.com/valentinmk/react-redux-universal-hot-example/commit/510a59f732a9bf42e070e7f57e970a2307661739)


### Пользовательский интерфейс Вариант 2. Компоненты.

Давайте разделим все на компоненты. Ничего сложного.

Создаем новую папку в директории `src\components` назовем ее `SocketExampleComponents`.

Добавление компонента происходит в три шага

1 - создаем файл с компонентом в нашей папке `SocketConnectionLog.js`

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

2 - прописываем наш новый компонент в файле `components\index.js`

```js
export SocketConnectionLog from './SocketExampleComponents/SocketConnectionLog';
```

3 - правим нашу страницу и вместо скопированного нами кода вставляем только один элемент

```js
// src/components/SocketExamplePage.js
// ...
import {SocketConnectionLog} from 'components';
// ...
      <SocketConnectionLog />
```

Добавляем другой новый компонент в ту же папку `src\components\SocketExampleComponents`.

Добавляем в три шага
1 - создаем файл с компонентом в нашей папке `SocketMessageLog.js`

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
2 - прописываем наш новый компонент в файле `components\index.js`

```js
export SocketMessageLog from './SocketExampleComponents/SocketMessageLog';
```

3 - правим нашу страницу и вместо скопированного нами кода вставляем только один элемент

```js
// src/components/SocketExamplePage.js
// ...
import {SocketMessageLog} from 'components';
// ...
        <SocketMessageLog/>
```

Проверяем. Ничего не изменилось - это успех.



Коммитимся и заканчиваем 2 часть.
