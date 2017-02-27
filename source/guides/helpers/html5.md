---
title: Руководство - Хелперы HTML5
---

## Хелперы HTML5

Эти хэлперы упрощают работу с HTML5. Они **не привязаны к конкретному шаблонизатору**.
Доступ к ним осуществляется как к **закрытым методам представлений** посредством вызова `#html`.

## Использование

Вот как их можно использовать в макете:

```ruby
module Web::Views
  class ApplicationLayout
    include Web::Layout

    def sidebar
      html.aside(id: 'sidebar') do
        div 'hello'
      end
    end
  end
end
```

```erb
<%= sidebar %>
```

Будет сгенерировано:

```html
<aside id="sidebar">
  <div>hello</div>
</aside>
```

## Особенности

  1) Метод закроет тэг согласно стандарту HTML5;
  2) Первый аргумент станет содержимым тэга;
  3) В качестве первого аргумента может быть передан аналогичный метод;
  4) В качестве первого аргумента может быть передан блок, который возвращает строку;
  5) Переданный блок может содержать в себе другие методы разметки;
  6) Атрибуты тэга передаются в качестве хэша;
  7) Атрибуты и блок могут использоваться вместе.

```ruby
# 1
html.div # => <div></div>
html.img # => <img>

# 2
html.div('hello') # => <div>hello</div>

# 3
html.div(html.p('hello')) # => <div><p>hello</p></div>

# 4
html.div { 'hello' }
# =>
#<div>
#  hello
#</div>

# 5
html.div do
  p 'hello'
end
# =>
#<div>
#  <p>hello</p>
#</div>

# 6
html.div('hello', id: 'el', 'data-x': 'y') # => <div id="el" data-x="y">hello</div>

# 7
html.div(id: 'yay') { 'hello' }
# =>
#<div id="yay">
#  hello
#</div>
```

It supports complex markup constructs, **without the need of concatenate tags**. In the following example, there are two `div` tags that we don't need link together.

```ruby
html.section(id: 'container') do
  div(id: 'main') do
    p 'Main content'
  end
  
  div do
    ul(id: 'languages') do
      li 'Italian'
      li 'English'
    end
  end
end

# =>
#  <section id="container">
#    <div id="main">
#      <p>Main Content</p>
#    </div>
#
#    <div>
#      <ul id="languages">
#        <li>Italian</li>
#        <li>English</li>
#      </ul>
#    </div>
#  </section>
```

В результате можно получить очень чистый API.

## Пользовательские тэги

Хэлперы Ханами поддерживают более 100 стандартных тэгов, таких как `div`, `video` и `canvas`.
Тем не менее, HTML5 продолжает развиваться и мы хотим иметь достаточно открытый интерфейс для работы с **новыми или пользовательскими тэгами**.

Соответствующий API очень прост: `#tag` создаст парный тэг, а `#empty_tag` одиночный.

```ruby
html.tag(:custom, 'Foo', id: 'next') # => <custom id="next">Foo</custom>
html.empty_tag(:xr, id: 'next')      # => <xr id="next">
```

## Другие хелперы

Хэлперы Ханами для HTML дают еще несколько возможностей. Например, хэлпер `link_to`:

```ruby
html.div do
  link_to 'hello', routes.root_path, class: 'btn'
end
# => <div>
# =>   <a href="/" class="btn">hello</a>
# => </div>

html.div do
  link_to 'Users', routes.users_path, class: 'btn'
  hr
  link_to 'Books', routes.books_path, class: 'btn'
end

# => <div>
# =>   <a href="/users" class="btn">Users</a>
# =>   </hr>
# =>   <a href="/posts" class="btn">Books</a>
# => </div>
```

## Экранирование

Тэги автоматически экранируются из соображений **безопасности**:

```ruby
html.div('hello')         # => <div>hello</div>
html.div { 'hello' }      # => <div>hello</div>
html.div(html.p('hello')) # => <div><p>hello</p></div>
html.div do
  p 'hello'
end # => <div><p>hello</p></div>



html.div("<script>alert('xss')</script>")
  # =>  "<div>&lt;script&gt;alert(&apos;xss&apos;)&lt;&#x2F;script&gt;</div>"

html.div { "<script>alert('xss')</script>" }
  # =>  "<div>&lt;script&gt;alert(&apos;xss&apos;)&lt;&#x2F;script&gt;</div>"

html.div(html.p("<script>alert('xss')</script>"))
  # => "<div><p>&lt;script&gt;alert(&apos;xss&apos;)&lt;&#x2F;script&gt;</p></div>"

html.div do
  p "<script>alert('xss')</script>"
end
  # => "<div><p>&lt;script&gt;alert(&apos;xss&apos;)&lt;&#x2F;script&gt;</p></div>"
```

**Атрибуты тэгов не экранируются автоматически**. В случаях, когда значения атрибутов вводятся пользователем, мы рекомендуем  использовать метод `#ha`, который создан как раз для этого. Больше информациии об экранирующих тэгах можно найти в [соответствующем разделе](`/guides/helpers/escape`).

## Контекст представлений

Локальные переменные представлений будут доступны внутри блоков, переданых хэлперам:

```ruby
module Web::Views::Books
  class Show
    include Web::View

    def title_widget
      html.div do
        h1 book.title
      end
    end
  end
end
```

```erb
<div id="content">
  <%= title_widget %>
</div>
```

```html
<div id="content">
  <div>
    <h1>The Work of Art in the Age of Mechanical Reproduction</h1>
  </div>
</div>
```

