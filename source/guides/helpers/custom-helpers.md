---
title: Руководство - Хэлперы: Пользовательские хэлперы 
---

## Пользовательские хэлперы

В [обзорном](/guides/helpers/overview) разделе мы кратко рассказали о предназначении и реализации хэлперов.
Мы говорили о них как о модулях, которые созданы для того, чтобы сделать код представлений более выразительным.
А так как они являются обыкновенными модулями Ruby, то мы можем **создать собственные хэлперы**.

### Пример

Представим, что нам необходим хэлпер, который перемешивает символы внутри строки. Мы хотим сделать его доступным на уровне представлений.

Для начала объявим модуль.

```ruby
# app/web/helpers/shuffler.rb
module Web
  module Helpers
    module Shuffler
      private
      SEPARATOR = ''.freeze

      def shuffle(string)
        string
          .encode(Encoding::UTF_8, invalid: :replace)
          .split(SEPARATOR).shuffle.join
      end
    end
  end
end
```

<p class="notice">
  Не существует соглашения, определяющего имена файлов и модулей.
  Мы можем сделать их такими, какими захотим.
</p>

После этого добавим его путь к другим путям модулей приложения, чтобы оно могло его загрузить.

На последнем этапе необходимо включить модуль для всех представлений. См. раздел [Общий код представлений](/guides/views/share-code).

```ruby
# apps/web/application.rb
module Web
  class Application < Hanami::Application
    configure do
      # ...

      load_paths << [
        'helpers',
        'controllers',
        'views'
      ]

      # ...

      view.prepare do
        include Hanami::Helpers
        include Web::Helpers::Shuffler
      end
    end
  end
end
```

<p class="notice">
  Обратите внимание, что наш созданный хэлпер будет работать даже если мы удалим строку <code>include Hanami::Helpers</code>.
</p>
