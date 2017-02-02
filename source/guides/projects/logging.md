---
title: Guides - Logging
---

# Ведение журнала

Каждый проект имеет свой логгер.

<p class="convention">
  Для приложения с названием <code>Web</code> логгер будет доступен как <code>Web.logger</code>.
</p>

Настройки логгера определяются с настройками окружения: их назначение вывода, формат и уровень детализации.

Например, по умолчанию логи направлены на стандартный поток вывода, но мы можем настроить вывод и в файл.

```ruby
# apps/web/application.rb
module Web
  class Application < Hanami::Application
    # ...

    configure :development do
      # ...

      # Logger
      # See: http://hanamirb.org/guides/projects/logging
      #
      # Направление вывода, по умолчанию STDOUT
      # logger.stream "log/development.log"
      #
      # Уровень детализации, по умолчанию DEBUG
      # logger.level :debug
      #
      # Формат, по умолчанию DEFAULT
      # logger.format :default
    end

    ##
    # TEST
    #
    configure :test do
      # ...

      # Logger
      # See: http://hanamirb.org/guides/projects/logging
      #
      # Уровень детализации, по умолчанию ERROR
      logger.level :error
    end

    ##
    # PRODUCTION
    #
    configure :production do
      # ...

      # Logger
      # See: http://hanamirb.org/guides/projects/logging
      #
      # Направление вывода, пo умолчанию STDOUT
      # logger.stream "log/production.log"
      #
      # Уровень детализации, по умолчанию INFO
      logger.level :info

      # Форма вывода
      logger.format :json
    end
  end
end
```

Использование стандартного потока вывода это [общепринятая практика](http://12factor.net/logs). Большинство компаний по предоставлению SaaS хостингов [предпочитают ей следовать](https://devcenter.heroku.com/articles/rails4#logging-and-assets).

Благодаря простоте обработки JSON является форматом по умолчанию для окружения, используемого в эксплуатации. С ним будет легко оценить результаты работы приложения.  


Логгер Hanami очень похож на обычный `Logger` Ruby. Простой пример использования:

```ruby
Web.logger.debug "Hello"
```
