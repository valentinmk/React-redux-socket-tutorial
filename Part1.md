# Часть первая. Настройка страницы

Для настройки раздела используем предоставленную справку от команды [react-redux-universal-hot-example](https://github.com/erikras/react-redux-universal-hot-example). Оригинал статьи находится [тут](https://github.com/erikras/react-redux-universal-hot-example/blob/master/docs/AddingAPage/AddingAPage.md).


### Создаем новый контейнер

```bash
cd ./src/containers && mkdir ./SocketExample
```

### Копируем туда hello.js как шаблон странички

```bash
cp About/About.js Hello/SocketExamplePage.js
```

Я использую для всего этого Atom, как действительно прекрасный редактор-чего-угодно с некоторыми плюшками.

###Правим скопированный файл

Создаем заглушку под нашу страница. Вводим элемент `<p>`. Позже будем выводить статус соединения в этот элемент.

```js
import React, {Component} from 'react';
import Helmet from 'react-helmet';

export default class SocketExamplePage extends Component {
  render() {
    return (
      <div className="container">
        <h1>Socket Exapmle Page</h1>
        <Helmet title="Socket Exapmle Page"/>
        <p>Sockets not connected</p>
      </div>
    );
  }
}```

### Подключаем

Добавляем в `./src/containers/index.js` новый компонент React

```js
export SocketExamplePage from './SocketExample/SocketExamplePage';
```

Добавляем в `./routes.js`, чтобы связать переход по пунти `/socketexamplepage` в карту ссылок

```js
...
import {
    App,
    Chat,
    Home,
    Widgets,
    About,
    Login,
    LoginSuccess,
    Survey,
    NotFound,
    SocketExamplePage
  } from 'containers';
...
      { /* Routes */ }
      <Route path="about" component={About}/>
      <Route path="login" component={Login}/>
      <Route path="survey" component={Survey}/>
      <Route path="widgets" component={Widgets}/>
      <Route path="socketexamplepage" component={SocketExamplePage}/>
...
```

Добавляем в `./src/containers/App/App.js`, чтобы добавить пункт в меню 

```js
              <LinkContainer to="/socketexamplepage">
                <NavItem eventKey={99}>Socket Example Page</NavItem>
              </LinkContainer>
```

Проверяем ```bash
npm run dev
```

На данный момент мы имеем:

* Раздел веб приложения
* Страничка на React для нашего приложения
* Заготовка, чтобы идти дальше







