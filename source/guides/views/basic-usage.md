---
title: Документация - Использование представлений
---

# Использование

В [предыдущем разделе](/guides/views/overview) мы создали представление. Попробуем его использовать.

## Формирование страниц по умолчанию

Для начала изменим сответствующий шаблон:

```erb
# apps/web/templates/dashboard/index.html.erb
<h1>Dashboard</h1>
```

Открыв в браузере `/dashboard` мы должны будем увидеть `<h1>Dashboard</h1>`.

Здесь мы снова сталкиваемся с соглашением о именовании.
Наше представление называется `Web::Views::Dashboard::Index`, а файл шаблона `web/templates/dashboard/index`.

<p class="convention">
  Для представления <code>Web::Views::Dashboard::Index</code> должен существовать шаблон <code>apps/web/templates/dashboard/index.html.erb</code>.
</p>

### Контекст

Во время отрисовки файла шаблона используется _контекст представления_.

```erb
# apps/web/templates/dashboard/index.html.erb
<h1><%= title %></h1>
```

Наличие переменных в шаблоне соответствует их наличию в представлении.

```ruby
# apps/web/views/dashboard/index.rb
module Web::Views::Dashboard
  class Index
    include Web::View

    def title
      'Dashboard'
    end
  end
end
```

В этом случае представление обеспечит доступ к `#title` внутри шаблона реализовав такой метод.
Мы снова увидим `<h1>Dashboard</h1>` на странице `/dashboard`.

### Данные из экшена

Контекст формируется также и [_выставлениями(exposures)_](/guides/actions/exposures).
Это те данные, которые передаются из экшена.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action
    expose :title

    def call(params)
      @title = 'Dashboard'
    end
  end
end
```

Мы можем убрать `#title` из представления и сделать его доступным другим способом.

```ruby
# apps/web/views/dashboard/index.rb
module Web::Views::Dashboard
  class Index
    include Web::View
  end
end
```

<p class="notice">
Контекст внутри шаблона определяется контекстом представления и выставлениями(exposures) экшена.
</p>

## Управление формированием страниц

Hanami формирует страницу при помощи вызова `#render` внутри представления и ожидает от него возврата строки.

Объектно-ориентированный подход позволяет легко изменить это поведение.

Мы можем просто переопределить этот метод.

```ruby
# apps/web/views/dashboard/index.rb
module Web::Views::Dashboard
  class Index
    include Web::View

    def render
      raw %(<h1>Dashboard</h1>)
    end
  end
end
```

Результат в браузере не изменится, но на этот раз шаблон не будет использован совсем.

<p class="convention">
Если в представлении переопределен метод <code>#render</code>, то он должен возвращать строку. Эта строка станет телом ответ и шаблон можно будет удалить, так как он перестанет использоваться.
</p>

## Пропуск представления

Если экшн явно использует определение тела ответа `#body=`, то представление будет [пропущено](/guides/actions/basic-usage).
