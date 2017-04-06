---
title: Руководство - Куки
---

# Куки

## Включение куки

Hanami следует философии _"аккумулятор идет в комплекте, но не установлен"_ .
Куки это одна из тех вещей, которые нужно включить.

Для этого необходимо просто раскомментировать строку.

```ruby
# apps/web/application.rb
module Web
  class Application < Hanami::Application
    configure do
      # ...
      cookies true
    end
  end
end
```

Теперь куки автоматически будут включены в ответ.

### Настройки

Используя эти настройки мы можем определить какие именно куки будет отправлять наше приложение.

  * `:domain` - `String` (по умолчанию `nil`), домен
  * `:path` - `String` (по умолчанию `nil`), относительный URL
  * `:max_age` - `Integer` (по умолчанию `nil`), срок годности куки в секундах
  * `:secure` - `Boolean` (по умолчанию `true` при использовании SSL), держит куки в пределах безопасного соединения
  * `:httponly` - `Boolean` (по умолчанию `true`), запрещает доступ JavaScript к куки

## Использование

Куки ведут себя как хэш: мы можем читать, устанавливать и удалять значения.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action

    def call(params)
      cookies[:b]         # прочитать
      cookies[:a] = 'foo' # установить
      cookies[:c] = nil   # удалить
    end
  end
end
```

Для установки значения можно использовать строки или хэш.
Основные настройки определяются автоматически, а указанные внутри экшена будут записаны поверх стандартных.

### Пример

```ruby
# apps/web/application.rb
module Web
  class Application < Hanami::Application
    configure do
      # ...
      cookies max_age: 300 # 5 минут
    end
  end
end
```

Мы собираемся установить две куки: первая использует конфигурацию приложения по умолчанию, а вторая использует переназначенные параметры.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action

    def call(params)
      # Set-Cookie:a=foo; max-age=300; HttpOnly
      cookies[:a] = 'foo'

      # Set-Cookie:b=bar; max-age=100; HttpOnly
      cookies[:b] = { value: 'bar', max_age: 100 }
    end
  end
end
```
