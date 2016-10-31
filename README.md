# Введение

Примерное содержание

## Используемые технологии

Redux - официальная документация расположена по адресу [http://redux.js.org](http://redux.js.org/). По-русски есть несколько вариантов, я лично использовал в основном [https://rajdee.gitbooks.io/redux-in-russian/content/docs/introduction/index.html](https://rajdee.gitbooks.io/redux-in-russian/content/docs/introduction/index.html).

Огромная благодарность товарищу exec64. Его статья стала тригерром к данному туториалу [https://exec64.co.uk/blog/websockets_with_redux/](https://exec64.co.uk/blog/websockets_with_redux/).


## Мотивация
Вообще я разрабатываю приложение на языке Python. Погодо-погоди уходить ... В рамках разработки я добрался до момента, когда сервер приложений уткнулся в отсутствие пользовательского интерфейса и требовала уже просто кричала о необходимости пользовательского взаимодействия.

Что мне нужно было:
* современные технологии (мне нечего было терять или быть чем то обязанным "старым проверенным приемам")
* это должно быть одностраничное приложений
* мне нужна реакция пользовательского интерфейса в реальном времени
* мне нужен обмен информацией сервер-клиент в реальном времени
* мне нужна возмонжость генерировать обновления клиента на сервере

Что было испробавано: 
* вариации на тему на чистом js (устарело)
* JQuery (уже не могу ТАК извратить так свой мозг)
* Angular (переход на 2 версию спугнул)
* Socket.io (слишком сильно привязывает серверную часть на node, мне нужен только клиент)

Выбрано в итоге:
* React (внятно/просто + babel = почти съедобно)
* Redux (импонирует использование ~~единой помойки~~ единого хранилища)
* WebSockets (очень просто и не связывает руки и фантазию)


## Сожержание

* [Часть первая. Настройка страницы](https://valentinmk.gitbooks.io/react-redux-socket-tutorial/content/Part1.html)
* [Часть вторая. Проектирование будущего приложения](https://valentinmk.gitbooks.io/react-redux-socket-tutorial/content/Part2.html)
* [Часть третья. Изумительный Redux](https://valentinmk.gitbooks.io/react-redux-socket-tutorial/content/Part3.html) 
* [Часть четвертая. Оживляем историю](https://valentinmk.gitbooks.io/react-redux-socket-tutorial/content/Part4.html)
* [Часть пятая. Проектируем чат](https://valentinmk.gitbooks.io/react-redux-socket-tutorial/content/Part5.html)
* [Часть шестая. Мидлваре](https://valentinmk.gitbooks.io/react-redux-socket-tutorial/content/Part6.html)
* [Часть седьмая. Все работает вместе](https://valentinmk.gitbooks.io/react-redux-socket-tutorial/content/Part7.html)
* [Часть последняя. Правы ли мы. Работа над ошибками](https://valentinmk.gitbooks.io/react-redux-socket-tutorial/content/FinalPart.html)



This file file serves as your book's preface, a great place to describe your book's content and ideas.

VK


(c) Valentin Kazakov 2016