---
title: Документация - MIME типы
---

# MIME типы

В экшенах есть продвинутые функции, работающие с MIME: определение, обработка заголовков, белый список и др..

## Анализ запросов

Чтобы определить тип MIME, экшн анализирует заголовок запроса `Accept` и предлагает высокоуровневый API: `#format` и `#accept?`.

Первый метод возвращает символ, отражающий MIME тип (`:html`, `:json`, `:xml` и др.), а второй делает запрос и проверяет, поддерживает ли данный браузер соответствующий тип.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action

    def call(params)
      puts format                     # => :html

      puts accept?('text/html')       # => true
      puts accept?('application/png') # => false
    end
  end
end
```

## Автоматическое определение Content-Type

Экшн определяет заголовок ответа `Content-Type` автоматически исходя из MIME типа запроса и его кодировки.

Если клиент пришлет запрос `Accept: text/html,application/xhtml+xml,application/xml;q=0.9`, то экшн вернет `Content-Type: text/html; charset=utf-8`.

### Формат запроса по умолчанию

Если клиент делает обобщенный запрос `Accept: */*`, экшн будет использовать **формат по умолчанию**.
Эта настройка позволяет нам безопасно обрабатывать подобные запросы. По умолчанию ее значение `:html`.

```ruby
# apps/web/application.rb

module Web
  class Application < Hanami::Application
    configure do
      # ...
      default_request_format :json
    end
  end
end
```

### Формат ответа по умолчанию

Если мы создаем приложение, являющееся JSON API, то нам может пригодиться настройка `:json` в качестве формата ответа по умолчанию. По умолчанию ее значение `:html`.

```ruby
# apps/web/application.rb

module Web
  class Application < Hanami::Application
    configure do
      # ...
      default_response_format :json
    end
  end
end
```

### Кодировка по умолчанию

Аналогично мы можем указать разные кодировки.
По умолчанию установлена `utf-8`, но ее мы можем так же изменить в файле конфигурации.

```ruby
# apps/web/application.rb

module Web
  class Application < Hanami::Application
    configure do
      # ...
      default_charset 'koi8-r'
    end
  end
end
```

### Переназначение

Существует способ указать возвращаемый `Content-Type` используя `#format=`.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action

    def call(params)
      puts self.format # => :html

      # force a different value
      self.format        =  :json
      puts self.format # => :json
    end
  end
end
```

Пример выше вернет `Content-Type: application/json; charset=utf-8`.

## Белый список

Мы можем ограничить диапазон MIME типов, которые могут быть приняты.
Если запрос не удовлетворяет этим ограничениям, то приложение вернет `Not Acceptable` статус (`406`).

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action
    accept :html, :json

    def call(params)
      # ...
    end
  end
end
```

## Регистрация MIME типов

Hanami распознает более 100 наиболее популярных MIME типов.
При этом мы можем захотеть добавить свой тип и использовать его с `#format=` и `.accept`.

В настройках нашего приложения мы можем использовать `controller.format`, который принимает хэш, где ключем является символ с именем формата (`:custom`), а значение строкой определяющей стандарт MIME типа (`application/custom`).

```ruby
# apps/web/application.rb

module Web
  class Application < Hanami::Application
    configure do
      # ...
      controller.format custom: 'application/custom'
    end
  end
end
```
