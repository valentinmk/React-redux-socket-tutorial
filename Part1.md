# Часть первая. Первоначальная настройка. Настройка страницы.

## Настройка окружения
Нам нужен node.js и npm.

Ставим node.js с сайта [https://nodejs.org](https://nodejs.org) - а именно этот гайд написан на 6ой версии, версию 7 тестировал - все работает.
npm устанавливается вместе с node.js

Далее нужно запустить npm и обновить node.js (для windows все тоже самое без npm)

```bash
sudo npm cache clean -f
sudo npm install -g n
sudo n stable
```
проверяем
```bash
node -v
```

## Настройка react-redux-universal-hot-example

Все выложено в [react-redux-universal-hot-example](https://github.com/erikras/react-redux-universal-hot-example), там же инструкция по установке.
Тут привожу последовательность действий

1. Скачиваем и разархивируем архив/форкаем/что-угодно-как-вам-нравится.
2. Через node.js command line или терминал переходим в эту папку
3. Запускаем







## Создаем новый контейнер

Для настройки раздела используем предоставленную справку от команды [react-redux-universal-hot-example](https://github.com/erikras/react-redux-universal-hot-example). Оригинал статьи находится [тут](https://github.com/erikras/react-redux-universal-hot-example/blob/master/docs/AddingAPage/AddingAPage.md).


```bash
cd ./src/containers && mkdir ./SocketExample
```

### Копируем туда hello.js как шаблон странички

```bash
cp About/About.js Hello/SocketExamplePage.js
```

Я использую для всего этого Atom, как действительно прекрасный редактор-чего-угодно с некоторыми плюшками.

##Правим скопированный файл

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
