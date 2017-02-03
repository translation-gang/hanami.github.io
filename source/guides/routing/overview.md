---
title: Guides - Routing Overview
---

# Обзор

Для маршрутизации приложения Hanami используют [Hanami::Router](https://github.com/hanami/router). Это легковесый и быстрый HTTP роутер совместимый с Rack.  

## Введение

Откроем в текстовом редакторе файл проекта `apps/web/config/routes.rb` и добавим следущие строки.

```ruby
get '/hello', to: ->(env) { [200, {}, ['Привет от Hanami!']] }
```

Затем запустим сервер при помощи команды `bundle exec hanami server` и откроем в браузере [http://localhost:2300/hello](http://localhost:2300/hello). В нем вы должны будете увидеть текст `Привет от Hanami!`.

Давайте разберемся в том, что мы только что сделали.

Мы создали **маршрут(route)**. Каждый маршрут начинается с [HTTP глагола](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html). В нашем случае `get`.
Затем необходимо указать относительный URI, в нашем случае `/hello`. После этого объект станет отвечать на соответствующие запросы к серверу.

В маршрутах необходимо использовать стандартные HTTP глаголы: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `TRACE` и `OPTIONS`.

```ruby
endpoint = ->(env) { [200, {}, ['Привет от Hanami!']] }

get     '/hello', to: endpoint
post    '/hello', to: endpoint
put     '/hello', to: endpoint
patch   '/hello', to: endpoint
delete  '/hello', to: endpoint
trace   '/hello', to: endpoint
options '/hello', to: endpoint
```

## Действия(экшены)

Выше мы увидели хорошую иллюстрацию интеграции с Rack, но в веб-приложениях в качестве точки назначения чаще используются **действия(экшены)**.
Действия — это объекты, ответственные за обработку HTTP запросов.
Как правило они доступны под именем похожим на `Web::Controllers::Home::Index`. Этот способ именования избыточен, поэтому Hanami использует **соглашение по именованию**, которое позволяет превратить эту длинную запись в такую: `"home#index"`.

```ruby
# apps/web/config/routes.rb
root to: "home#index" # => аналог маршрута к Web::Controllers::Home::Index
```
Первая часть этого выражения это имя контроллера, `"home"` воспринимается как `Home`.
Аналогичные преобразования произойдут и со второй частью после `#`: `"index"` станет `Index`.

Hanami позволяет описать маршрут кратко при помощи пространства имен по умолчанию (`Web::Controllers`)  или использовать полное имя класса.

## Rack

Hanami полностью совместим с [Rack SPEC](http://www.rubydoc.info/github/rack/rack/master/file/SPEC). Это позволяет гибко работать с пунктами назначения.
В следующем примере мы использовали объекты `Proc`.
В качестве пункта назначения может выступать любой объект, класс, действие или **приложение**, реализующее метод `#call`.

```ruby
get '/proc',       to: ->(env) { [200, {}, ['Hello from Hanami!']] }
get '/action',     to: "home#index"
get '/middleware', to: Middleware
get '/rack-app',   to: RackApp.new
get '/rails',      to: ActionControllerSubclass.action(:new)
```
Когда мы передаем строку, произойдет попытка получить из нее экземпляр класса:

```ruby
get '/rack-app', to: 'rack_app' # преобразуется в RackApp.new
```

### Подключение приложения

Когда необходимо подключить приложение используется `mount`.

#### Подключение при помощи пути

```ruby
mount SinatraApp.new, at: '/sinatra'
```
Теперь все HTTP-запросы, которые начинаются с `/sinatra` будут направлены к `SinatraApp`.

#### Подключение к поддомену

```ruby
mount Blog.new, host: 'blog'
```

Таким образом все запросы к `http://blog.example.com` будут направлены к `Blog`.

<p class="notice">
  Во время разработки у вас не будет доступа к <code>http://blog.localhost:2300</code>,
  поэтому необходимо уточнить host во время запуска сервера:
  <code>bundle exec hanami server --host=lvh.me</code>.
  Тогда во время разработки вы сможете получить доступ к <code>http://blog.lvh.me:2300</code>
</p>
