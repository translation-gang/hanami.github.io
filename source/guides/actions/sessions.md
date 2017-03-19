---
title: Руководство - Экшены: Сессии
---

# Сессии

## Включение сессий

Ханами поддерживает сессии, но не включает их по умолчанию.
Если мы хотим использовать их, то можем просто раскомментировать следующую строку.

```ruby
# apps/web/application.rb
module Web
  class Application < Hanami::Application
    configure do
      # ...
      sessions :cookie, secret: ENV['WEB_SESSIONS_SECRET']
    end
  end
end
```

Первым аргументом передается имя адаптера для хранения сессий.
По умолчанию установлено значение `:cookie`, использующее `Rack::Session::Cookie`.

<p class="convention">
В качестве имени адаптера сессий используется имя класса из пространства имен <code>Rack::Session</code> записанное строчными буквами с подчеркиваниями.
Например: <code>:cookie</code> для <code>Rack::Session::Cookie</code>.
</p>

Мы можем использовать другие хранилища, совместимые с сессиями Rack.
Предположим, мы хотим использовать Redis. Мы должны подключить к проекту `redis-rack` и указать имя адаптера `:redis`.
Ханами автоматически подключит адаптер во время загрузки приложения.

<p class="convention">
Нестандартные хранилища будут автоматически загружаться через <code>require "rack/session/#{ имя адаптера }"</code>.
</p>

Второй аргумент принимаемый `sessions` это хэш параметров, которые будут **переданы адаптеру**.
По умолчанию передается `:secret`, но мы можем использовать любые параметры, которые принимает адаптер.

## Использование

Сессии ведут себя как хэш: их можно читать, записывать или удалять их значения.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action

    def call(params)
      session[:b]         # чтение
      session[:a] = 'foo' # запись
      session[:c] = nil   # удаление
    end
  end
end
```
