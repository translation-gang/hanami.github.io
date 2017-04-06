---
title: Руководство - Экшены: Интеграция с Rack
---

# Интеграция с Rack

## Rack окружение

Экшены(actions) просто предоставляют высокоуровневый API для Rack.
Если мы захотим получить доступ к исходным данным Rack окружения, то можем использовать `params.env`.

## Промежуточное ПО Rack

Ханами по умолчанию использует очень тонкий промежуточный слой(middleware) для своих приложений.
Дополнительные компоненты могут быть подключены глобально, на уровне приложения или локально.

### Промежуточное ПО глобального уровня

Когда необходим компонент, который будет использоваться всеми приложениями (всеми в каталоге `apps/`), мы можем изменить `config.ru` в корневом каталоге проекта.

```ruby
# config.ru
require './config/environment'
require 'rack/auth/basic'

use Rack::Auth::Basic
run Hanami.app
```

### Промежуточное ПО уровня приложения

Когда необходим компонент, который будет использоваться отдельным приложением (одним из `apps/`), мы можем изменить конфигурацию самого приложения.

```ruby
# apps/web/application.rb
require 'rack/auth/basic'

module Web
  class Application < Hanami::Application
    configure do
      # ...
      middleware.use Rack::Auth::Basic
    end
  end
end
```

### Промежуточное ПО уровня экшена

Иногда необходимо промежуточное ПО, которое будет использовано для отдельных ресурсов.
Если подключить его глобально, то производительность приложения неоправданно упадет.
Экшены же позволят нам тонко настроить набор промежуточного ПО.

```ruby
# apps/web/controllers/sessions/create.rb
require 'omniauth'

module Web::Controllers::Sessions
  class Create
    include Web::Action

    use OmniAuth::Builder {
      # ...
    }

    def call(params)
      # ...
    end
  end
end
```

Такой синтаксис применяется для подключения промежуточного ПО, которое принимает дополнительные аргументы.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action

    use XMiddleware.new('x', 123)
    use YMiddleware.new
    use ZMiddleware

    def call(params)
      # ...
    end
  end
end
```
