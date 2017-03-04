---
title: Руководство - Разделение кода представлений
---

# Общий код

## Метод prepare

В настройках приложения(`apps/web/application.rb`) есть участок кода, разделяемый **всеми представлениями** в нашем приложении. 
Когда любое представление подключит `Web::View`, будет вызван этот участок кода в контексте данного представления.

Этот метод основан на механизме работы Ruby модулей и называется обратным вызовом `included`.

Представим, что у нас есть приложение, которое рендерит только JSON.
Для каждого представления нам придется указать формат. Добавлять эту строку в каждый файл вручную было бы утомительно, поэтому мы последуем принципу DRY.

Создадим модуль `apps/web/views/accept_json.rb`.

```ruby
# apps/web/views/accept_json.rb
module Web::Views
  module AcceptJson
    def self.included(view)
      view.class_eval do
        format :json
      end
    end
  end
end
```

Затем включим его во все представления приложения сразу при помощи `view.prepare`.

```ruby
# apps/web/application.rb
module Web
  class Application < Hanami::Application
    configure do
      # ...
      view.prepare do
        include Web::Views::AcceptJson
      end
    end
  end
end
```

<p class="warning">
Код включенный в <code>prepare</code> доступен во всех представлениях приложения.
</p>
