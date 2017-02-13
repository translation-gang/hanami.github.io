---
title: Руководство - Ресурсы в стиле REST
---

# REST

В Hanami встроена поддержка [REST-архитектуры](http://en.wikipedia.org/wiki/Representational_state_transfer).

Для ее определения на уровне маршрутов существует два метода: `resources` и `resource`.

Определение ресурсов позволяет сгенерировать **несколько стандартных маршрутов** при помощи всего одной строки кода.

## Ресурсы в стиле REST

### Стандартные маршруты

```ruby
# apps/web/config/routes.rb
resources :books
```

Создаст

<table class="table table-bordered table-striped">
  <tr>
    <th>HTTP глагол</th>
    <th>Маршрут</th>
    <th>Экшн</th>
    <th>Имя экшена</th>
    <th>Имя маршрута</th>
  </tr>
  <tr>
    <td>GET</td>
    <td>/books</td>
    <td>Books::Index</td>
    <td>:index</td>
    <td>:books</td>
  </tr>
  <tr>
    <td>GET</td>
    <td>/books/:id</td>
    <td>Books::Show</td>
    <td>:show</td>
    <td>:book</td>
  </tr>
  <tr>
    <td>GET</td>
    <td>/books/new</td>
    <td>Books::New</td>
    <td>:new</td>
    <td>:new_book</td>
  </tr>
  <tr>
    <td>POST</td>
    <td>/books</td>
    <td>Books::Create</td>
    <td>:create</td>
    <td>:books</td>
  </tr>
  <tr>
    <td>GET</td>
    <td>/books/:id/edit</td>
    <td>Books::Edit</td>
    <td>:edit</td>
    <td>:edit_book</td>
  </tr>
  <tr>
    <td>PATCH</td>
    <td>/books/:id</td>
    <td>Books::Update</td>
    <td>:update</td>
    <td>:book</td>
  </tr>
  <tr>
    <td>DELETE</td>
    <td>/books/:id</td>
    <td>Books::Destroy</td>
    <td>:destroy</td>
    <td>:book</td>
  </tr>
</table>

### Удаление маршрутов

В случае когда нам нужны не все стандартные маршруты мы можем использовать параметр `:only` и задать одно или несколько имен экшенов.
Другой параметр, `:except`, позволяет аналогичным образом исключить маршруты из стандартного набора.

```ruby
resources :books, only: [:new, :create, :show]

# эквивалентно

resources :books, except: [:index, :edit, :update, :destroy]
```

### Добавление маршрутов

Вместе со стандартными маршрутами мы можем указать дополнительные как для одного экземпляра ресурса (`member`) так и для коллекции (`collection`).

```ruby
resources :books do
  member do
    # Создаст маршрут /books/1/toggle, направленный к Books::Toggle, именованный как :toggle_book
    get 'toggle'
  end

  collection do
    # Создаст маршрут /books/search, направленный к Books::Search, именованный как :search_books
    get 'search'
  end
end
```

### Явное указание контроллера

Допустим, у нас есть контроллер с именем `manuscripts`, в котором определен экшн `Manuscripts::Index`, но мы хотим сделать его доступным через `/books`.

Используя параметр `:controller` мы сэкономим много времени.

```ruby
resources :books, controller: 'manuscripts'

# GET /books/1 будет перенаправлен к Manuscripts::Show
```

## Ресурс в стиле REST

```ruby
resource :account
```

Создаст

<table class="table table-bordered table-striped">
  <tr>
    <th>HTTP глагол</th>
    <th>Маршрут</th>
    <th>Экшн</th>
    <th>Имя экшена</th>
    <th>Имя маршрута</th>
  </tr>
  <tr>
    <td>GET</td>
    <td>/account</td>
    <td>Account::Show</td>
    <td>:show</td>
    <td>:account</td>
  </tr>
  <tr>
    <td>GET</td>
    <td>/account/new</td>
    <td>Account::New</td>
    <td>:new</td>
    <td>:new_account</td>
  </tr>
  <tr>
    <td>POST</td>
    <td>/account</td>
    <td>Account::Create</td>
    <td>:create</td>
    <td>:account</td>
  </tr>
  <tr>
    <td>GET</td>
    <td>/account/edit</td>
    <td>Account::Edit</td>
    <td>:edit</td>
    <td>:edit_account</td>
  </tr>
  <tr>
    <td>PATCH</td>
    <td>/account</td>
    <td>Account::Update</td>
    <td>:update</td>
    <td>:account</td>
  </tr>
  <tr>
    <td>DELETE</td>
    <td>/account</td>
    <td>Account::Destroy</td>
    <td>:destroy</td>
    <td>:account</td>
  </tr>
</table>

### Удаление маршрутов

```ruby
resource :account, only: [:show, :edit, :update, :destroy]

# эквивалентно

resource :account, except: [:new, :create]
```

### Добавление маршрутов

```ruby
resource :account do
  member do
    # Создаст маршрут /account/avatar, направленный к Account::Avatar, именованный как :avatar_account
    get 'avatar'
  end

  collection do
    # Создаст маршрут /account/authorizations, направленный к Account::Authorizations, именованный как :authorizations_account
    get 'authorizations'
  end
end
```

### Явное указание контроллера

```ruby
resource :account, controller: 'customer'
```

## Вложенные ресурсы

Ресурсы в стиле REST могут быть вложены. Так они станут доступны как часть родительских ресурсов.

### Множественные ресурсы внутри множественных

```ruby
resources :books do
  resources :reviews
end
```

**Будут сгенерированы все стандартные пути для :books и вложений**

<table class="table table-bordered table-striped">
  <tr>
    <th>HTTP глагол</th>
    <th>Маршрут</th>
    <th>Экшн</th>
    <th>Имя экшена</th>
    <th>Имя маршрута</th>
  </tr>
  <tr>
    <td>GET</td>
    <td>/books/:book_id/reviews</td>
    <td>Books::Reviews::Index</td>
    <td>:index</td>
    <td>:book_reviews</td>
  </tr>
  <tr>
    <td>GET</td>
    <td>/books/:book_id/reviews/:id</td>
    <td>Books::Reviews::Show</td>
    <td>:show</td>
    <td>:book_review</td>
  </tr>
  <tr>
    <td>GET</td>
    <td>/books/:book_id/reviews/new</td>
    <td>Books::Reviews::New</td>
    <td>:new</td>
    <td>:new_book_review</td>
  </tr>
  <tr>
    <td>POST</td>
    <td>/books/:book_id/reviews</td>
    <td>Books::Reviews::Create</td>
    <td>:create</td>
    <td>:book_reviews</td>
  </tr>
  <tr>
    <td>GET</td>
    <td>/books/:book_id/reviews/:id/edit</td>
    <td>Books::Reviews::Edit</td>
    <td>:edit</td>
    <td>:edit_book_review</td>
  </tr>
  <tr>
    <td>PATCH</td>
    <td>/books/:book_id/reviews/:id</td>
    <td>Books::Reviews::Update</td>
    <td>:update</td>
    <td>:book_review</td>
  </tr>
  <tr>
    <td>DELETE</td>
    <td>/books/:book_id/reviews/:id</td>
    <td>Books::Reviews::Destroy</td>
    <td>:destroy</td>
    <td>:book_review</td>
  </tr>
</table>

### Единственный внутри множественного

```ruby
resources :books do
  resource :cover
end
```

**Будут сгенерированы все стандартные пути для :books и вложений**

<table class="table table-bordered table-striped">
  <tr>
    <th>HTTP глагол</th>
    <th>Маршрут</th>
    <th>Экшн</th>
    <th>Имя экшена</th>
    <th>Имя маршрута</th>
  </tr>
  <tr>
    <td>GET</td>
    <td>/books/:book_id/cover</td>
    <td>Books::Cover::Show</td>
    <td>:show</td>
    <td>:book_cover</td>
  </tr>
  <tr>
    <td>GET</td>
    <td>/books/:book_id/cover/new</td>
    <td>Books::Cover::New</td>
    <td>:new</td>
    <td>:new_book_cover</td>
  </tr>
  <tr>
    <td>POST</td>
    <td>/books/:book_id/cover</td>
    <td>Books::Cover::Create</td>
    <td>:create</td>
    <td>:book_cover</td>
  </tr>
  <tr>
    <td>GET</td>
    <td>/books/:book_id/cover/edit</td>
    <td>Books::Cover::Edit</td>
    <td>:edit</td>
    <td>:edit_book_cover</td>
  </tr>
  <tr>
    <td>PATCH</td>
    <td>/books/:book_id/cover</td>
    <td>Books::Cover::Update</td>
    <td>:update</td>
    <td>:book_cover</td>
  </tr>
  <tr>
    <td>DELETE</td>
    <td>/books/:book_id/cover</td>
    <td>Books::Cover::Destroy</td>
    <td>:destroy</td>
    <td>:book_cover</td>
  </tr>
</table>

### Множественные внутри единственного

```ruby
resource :account do
  resources :api_keys
end
```

**Будут сгенерированы все стандартные пути для :account и вложений**

<table class="table table-bordered table-striped">
  <tr>
    <th>HTTP глагол</th>
    <th>Маршрут</th>
    <th>Экшн</th>
    <th>Имя экшена</th>
    <th>Имя маршрута</th>
  </tr>  
  <tr>
    <td>GET</td>
    <td>/account/api_keys</td>
    <td>Account::ApiKeys::Index</td>
    <td>:index</td>
    <td>:account_api_keys</td>
  </tr>
  <tr>
    <td>GET</td>
    <td>/account/api_keys/:id</td>
    <td>Account::ApiKeys::Show</td>
    <td>:show</td>
    <td>:account_api_key</td>
  </tr>
  <tr>
    <td>GET</td>
    <td>/account/api_keys/new</td>
    <td>Account::ApiKeys::New</td>
    <td>:new</td>
    <td>:new_account_api_key</td>
  </tr>
  <tr>
    <td>POST</td>
    <td>/account/api_keys</td>
    <td>Account::ApiKeys::Create</td>
    <td>:create</td>
    <td>:account_api_keys</td>
  </tr>
  <tr>
    <td>GET</td>
    <td>/account/api_keys/:id/edit</td>
    <td>Account::ApiKeys::Edit</td>
    <td>:edit</td>
    <td>:edit_account_api_key</td>
  </tr>
  <tr>
    <td>PATCH</td>
    <td>/account/api_keys/:id</td>
    <td>Account::ApiKeys::Update</td>
    <td>:update</td>
    <td>:account_api_key</td>
  </tr>
  <tr>
    <td>DELETE</td>
    <td>/account/api_keys/:id</td>
    <td>Account::ApiKeys::Destroy</td>
    <td>:destroy</td>
    <td>:account_api_key</td>
  </tr>
</table>

### Единственный внутри единственного

```ruby
resource :account do
  resource :avatar
end
```

**Будут сгенерированы все стандартные пути для :account и вложений**

<table class="table table-bordered table-striped">
  <tr>
    <th>HTTP глагол</th>
    <th>Маршрут</th>
    <th>Экшн</th>
    <th>Имя экшена</th>
    <th>Имя маршрута</th>
  </tr>
  <tr>
    <td>GET</td>
    <td>/account/avatar</td>
    <td>Account::Avatar::Show</td>
    <td>:show</td>
    <td>:account_avatar</td>
  </tr>
  <tr>
    <td>GET</td>
    <td>/account/avatar/new</td>
    <td>Account::Avatar::New</td>
    <td>:new</td>
    <td>:new_account_avatar</td>
  </tr>
  <tr>
    <td>POST</td>
    <td>/account/avatar</td>
    <td>Account::Avatar::Create</td>
    <td>:create</td>
    <td>:account_avatar</td>
  </tr>
  <tr>
    <td>GET</td>
    <td>/account/avatar/edit</td>
    <td>Account::Avatar::Edit</td>
    <td>:edit</td>
    <td>:edit_account_avatar</td>
  </tr>
  <tr>
    <td>PATCH</td>
    <td>/account/avatar</td>
    <td>Account::Avatar::Update</td>
    <td>:update</td>
    <td>:account_avatar</td>
  </tr>
  <tr>
    <td>DELETE</td>
    <td>/account/avatar</td>
    <td>Account::Avatar::Destroy</td>
    <td>:destroy</td>
    <td>:account_avatar</td>
  </tr>
</table>
