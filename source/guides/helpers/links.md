---
title: Guides - Link Helpers
---

## Link Helpers

Ханами предостовляет лаконичный API для создания ссылок.
Основной публичный метод - `#link_to`, который может быть использован как во вью объектах, так и в темплейтах.

## Использование

Этот метод принимает два обязательных и один опциональный аргумент.
Первый - это непосредственно текст ссылки (контент `<a>` тега), а второй - путь или нужная ссылка. Последний аргумент является хешом опций и он может принимать набор HTML аттрибутов которые мы хотим определить.

```erb
<%= link_to 'Главная', '/' %>
<%= link_to 'Профиль', routes.profile_path, class: 'btn', title: 'Ваш профиль' %>
<%=
  link_to(routes.profile_path, class: 'avatar', title: 'Ваш профиль') do
    img(src: user.avatar.url)
  end
%>
```

Код выше сгенерирует следующий HTML:

```html
<a href="/">Главная</a>
<a href="/profile" class="btn" title="Ваш профиль">Профиль</a>
<a href="/profile" class="avatar" title="Ваш профиль">
  <img src="/images/avatars/23.png">
</a>
```

В другом случае, содержимое тега может быть выражено в качестве блока.

```ruby
module Web::Views::Books
  class Show
    include Web::View

    def look_inside_link
      url = routes.look_inside_book_path(id: book.id)

      link_to url, class: 'book-cover' do
        html.img(src: book.cover_url)
      end
    end
  end
end
```

Темплейт файл:

```erb
<%= look_inside_link %>
```

Сгенерированный HTML:

```html
<a href="/books/1/look_inside" class="book-cover">
  <img src="https://cdn.bookshelf.org/books/1/full.png">
</a>
```

## Безопасность

Есть два аспекта, которые стоит учитывать, когда вы используете генерацию ссылок в разметке.
Это **белый список ссылок** и **экранированние аттирбутов**.
Пожалуйста, посмотрите раздел про [экранированную разметку](/guides/helpers/escape) для подробного объяснения.
