---
title: Руководство - Хелперы маршрутов
---

## Хэлперы маршрутов

Хэлперы маршрутов доступны через единственный **открытый метод**(`#routes`), доступный в экшенах, представлениях и шаблонах.
Это фабрика, генерирующая **относительные** и **абсолютные** URL, основываясь на [именованных маршрутах](/guides/routing/basic-usage).

<p class="convention">
  Для маршрута именованного как <code>:home</code> можно использовать <code>home_path</code> или <code>home_url</code> чтобы сгенерировать относительный или абсолютный URL.
</p>

## Использование

Представим, что у нас в приложении есть следующие маршруты:

```ruby
# web/apps/config/routes.rb
root        to: 'home#index'
get '/foo', to: 'foo#index'

resources :books
```

### Относительные URL

Мы можем написать:

```erb
<ul>
  <li><a href="<%= routes.root_path %>">Home</a></li>
  <li><a href="<%= routes.book_path %>">Books</a></li>
</ul>
```

Чтобы сгенерировать:

```html
<ul>
  <li><a href="/">Home</a></li>
  <li><a href="/books">Books</a></li>
</ul>
```

При этом мы не можем поступить так же с `/foo`, т.к. он не является именованным маршрутом(у него отсутствует параметр `:as`).

### Абсолютные URL

```ruby
module Web::Controllers::Books
  class Create
    include Web::Action

    def call(params)
      # ...
      redirect_to routes.book_url(id: book.id)
    end
  end
end
```
В примере выше мы дополнительно передали Хэш, при помощи которого будет сгенерирован URL для отдельного экземпляра ресурса.