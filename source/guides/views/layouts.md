---
title: Рукодство - Макеты
---

# Макеты

Макетами(layouts) называют такие файлы представлений, которые ответственны за рендеринг повторяющихся частей HTML страницы.
Часто они включают в себя хидэр, футер, меню навигации и подобные служебные элементы.

Во время генерации веб-приложения по умолчанию создается макет с названием `Web::Views::ApplicationLayout` и шаблоном `apps/web/templates/application.html.erb`.

Он будет содержать простой HTML5 каркас.

```erb
<!DOCTYPE HTML>
<html>
  <head>
    <title>Web</title>
  </head>
  <body>
    <%= yield %>
  </body>
</html>
```

Интереснее всего здесь строка `<%= yield %>`.
Во время выполнения она будет заменена на то, что вернет представление.
**Сначала рендерится представление, а затем макет.**

Контекст макета содержит в себе контекст текущего представления.
При этом контекст представления может переназначить собственный контекст макета.

Представим, что у нас есть строка `<title><%= page_title %></title>`.
Если макет и представление реализуют каждый свой `page_title`, то Hanami использует реализацию представления.

## Конфигурация макета

В конфигурации приложения по умолчанию содержится конфигурация макета.

```ruby
# apps/web/application.rb
module Web
  class Application < Hanami::Application
    configure do
      layout :application
    end
  end
end
```

<p class="convention">
Hanami добавляет к имени макета из конфигурации окончание <code>Layout</code>. Например, <code>layout :application</code> будет превращен в <code>Web::Views::ApplicationLayout</code>.
</p>

Если мы не хотим использовать макет, то можем его отключить так:

```ruby
# apps/web/views/dashboard/index.rb
module Web::Views::Dashboard
  class Index
    include Web::View
    layout false
  end
end
```

Если мы хотим отключить макеты полностью, то можем использовать `layout nil` в конфигурации приложения.

## Несколько макетов

Иногда бывает полезно иметь несколько макетов сразу.
Например, если `application.html.erb` содержит блок навигации и мы не хотим видеть его на странице авторизации, то решить проблему можно создав новый шаблон макета `login.html.erb`.


Допустим, у нас уже есть экшн `Web::Actions::UserSessions::New`, тогда мы можем создать `login.html.erb` рядом с основным макетом `application.html.erb` в папке `apps/web/templates/`.

Затем потребуется создать класс `Web::Views::LoginLayout`, который будет использовать шаблон нового макета. Его файл может называться `app/web/views/login_layout.rb`(в той же папке, что и `application_layout.rb`):

```ruby
module Web
  module Views
    class LoginLayout
      include Web::Layout
    end
  end
end
```

Теперь в экшене `app/web/views/user_sessions/new.rb` можно указать, какой макет мы хотим использовать в этом представлении:

```ruby
module Web::Views::UserSessions
  class New
    include Web::View
    layout :login
  end
end
```

К тому же, мы сможем использовать `layout :login` в любом другом представлении приложения.

## Необязательное содержимое

Иногда полезно рендерить содержимое не для всех страниц приложения.
Например, когда в страницу встроен специфичный javascript.

Если в качестве макета дан подобный шаблон:

```erb
<!DOCTYPE HTML>
<html>
  <!-- ... -->
  <body>
    <!-- ... -->
    <footer>
      <%= local :javascript %>
    </footer>
  </body>
</html>
```

С таким представлением:

```ruby
module Web::Views::Books
  class Index
    include Web::View
  end
end
```

И другим:

```ruby
module Web::Views::Books
  class Show
    include Web::View

    def javascript
      raw %(<script src="/path/to/script.js"></script>)
    end
  end
end
```

Первое представление ничего не будет знать о методе `#javascript`, оно без последствий его проигнорирует.
Второе же (`Web::Views::Books::Show`) будет отвечать на этот метод, а возвращенный результат станет содержимым страницы
