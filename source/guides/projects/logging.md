---
title: Руководство - Проекты: Логирование
---

# Логирование

Каждый проект имеет свой логер.

<p class="convention">
  Для приложения с названием <code>Web</code> логер будет доступен как <code>Web.logger</code>.
</p>

Настройки логера определяются настройками окружения: их назначение вывода, формат и уровень детализации.

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

Благодаря простоте обработки JSON является форматом по умолчанию для окружения, используемого в окружении эксплуатации(production). С ним будет просто оценить результаты работы приложения.  

Логер Ханами очень похож на обычный `Logger` Руби. Простой пример использования:

```ruby
Web.logger.debug "Hello"
```
