---
title: Руководство - Ассеты, обзор
---

# Ассеты

Ханами предоставляет ряд инструментов для работы с ассетами: таблицами стилей, сценариями JavaScript, изображениями и др..

## Конфигурация

Каждое приложение проекта имеет собственную конфигурацию для ассетов.

### Режим компиляции

Эта опция определяет, будет ли приложение обрабатывать ассеты или просто копировать их в папку доступную клиентам.
По умолчанию она включена в режиме _разработки_ и _тестирования_, но отключена в режиме _эксплуатации_.

```ruby
# apps/web/application.rb
module Web
  class Application < Hanami::Application
    configure do
      # ...
      assets do
        # compile true, по умолчанию опция включена
      end
    end

    configure :production do
      assets do
        compile false
      end
    end
  end
end
```

### Контрольные суммы

Чтобы браузеры правильно кэшировали файлы, во время развертывания приложения Ханами создает копию каждого файла и дополняет их имена [контрольными суммами](/guides/command-line/assets).

Мы можем отключить эту функцию в конфигурации приложения.
По умолчанию она включена только в режиме _эксплуатации_.

```ruby
# apps/web/application.rb
module Web
  class Application < Hanami::Application
    configure do
      # ...
      assets do
        # fingerprint false, опция отключена по умолчанию
      end
    end

    configure :production do
      assets do
        fingerprint true
      end
    end
  end
end
```

[Хэлперы ассетов](/guides/helpers/assets), если они включены, учтут контрольные суммы при создании относительного пути файла.

```erb
<%= javascript 'application' %>
```

```html
<script src="/assets/application-d1829dc353b734e3adc24855693b70f9.js" type="text/javascript"></script>
```

## Работа с статичными файлами

Статичные файлы во время разработки обслуживаются наравне с динамическими.
Для этого подключается специальный слой ПО промежуточного уровня `Hanami::Static`. Этот компонент подключается когда переменная окружения `SERVE_STATIC_ASSETS` равна `true`.

По умолчанию новые проекты включают эту функцию в режиме _разработки_ и _тестирования_ посредством файлов окружения `.env.*`.

```
# .env.development
# ...
SERVE_STATIC_ASSETS="true"
```

Ханами предполагает, что в режиме _эксплуатации_ проект будет использовать веб-сервер, такой как Nginx, который будет быстрее обслуживать статичные файлы не вызывая лишний раз код на Руби.

<p class="convention">
  Статичные файлы по умолчанию обслуживаются в режиме <em>разработки</em> и <em>тестирования</em>, но не в режиме <em>эксплуатации</em>.
</p>

Иногда это предположение оказывается неверным. Например, Heroku требует обслуживать статичные файлы при помощи Руби.
Чтобы включить эту функцию в режиме эксплуатации просто установите вышеописанную переменную окружения как `true` (в файле `.env.production`).

### Как Ханами обслуживает статичные файлы?

Как уже было сказано, когда соответствующая функция включена, промежуточный слой ПО `Hanami::Static` устанавливается в контейнер Rack.

Тогда входящие запросы могут быть обработаны по следующим сценариям.

#### "Свежие" ассеты

```
GET /assets/application.js
```

Файл `apps/web/assets/javascripts/application.js` будет скопирован в `public/assets/application.js` и вложен в ответ на запрос.

<p class="notice">
  Ассеты будут скопированы только если файла не существует по заданному пути(например: <code>public/assets/application.js</code>).
  Если файл существует, то он будет сразу же вложен в ответ.
</p>

<p class="warning">
  Когда в приложении отключена компиляция ассетов, Ханами не будет копировать файлы.
</p>

#### "Несвежие" ассеты

Это то, что происходит в _режиме разработки_, когда мы в первый раз запрашиваем ассет. Файл копируется в папку `public/`, а когда мы редактируем оригинал, копия становится "несвежей".

```
GET /assets/application.js
# отредактируем файл: apps/web/assets/javascripts/application.js
# и запросим его еще раз
GET /assets/application.js
```

Тогда файл будет **скопирован еще раз** в `public/assets/application.js` и передан в ответе.

#### Прекомпилированные ассеты

Допустим, мы хотим использовать для таблиц стилей препроцессор Sass.

```
GET /assets/application.css
```

Тогда из файла `apps/web/assets/stylesheet/application.css.sass` будет создан и передан в ответе файл `public/assets/application.css`.

#### Динамические ресурсы

```
GET /books/23
```

Это не статический файл, расположенный в папке `public/`, а значит управление будет передано в соответствующий экшн.

#### Несуществующий ресурс

```
GET /unknown
```

Это и не статический файл, и не динамический ресурс, а значит приложение ответит `404 (Not Found)`.

## Место хранения 

Каждое приложение имеет собственный набор папок, в которых оно будет искать файлы.
Поиск ассетов будет происходить рекурсивно в этих папках.

Новое приложение по умолчанию включает такую папку:

```
% tree apps/web/assets
apps/web/assets
├── favicon.ico
├── images
├── javascripts
└── stylesheets

3 папки, 1 файл
```

В ней мы можем расположить любое количество других папок(таких как: `apps/web/assets/fonts`).

<p class="convention">
  Приложение с именем <code>Web</code>, по умолчанию хранит ассеты в <code>apps/web/assets</code>
</p>

### Пользовательские пути хранения

Если мы захотим добавить новый путь хранения для отдельного приложения, то можем указать его в конфигурации приложения.

```ruby
# apps/web/application.rb
module Web
  class Application < Hanami::Application
    configure do
      # ...
      assets do
        # apps/web/assets включен по умолчанию
        sources << [
          'vendor/assets'
        ]
      end
    end
  end
end
```

Вышеописанный код добавит путь `apps/web/vendor/assets` и все его содержимое в список мест для поиска ассетов.

<p class="warning">
  Ханами проводит рекурсивный поиск в местах хранения ассетов. Пожалуйста, не храните в этих папках файлы, которые не должны быть доступны пользователю.
</p>

## Сторонние библиотеки

Ханами позволяет использовать [Rubygems](https://rubygems.org) чтобы расширить возможности использования ассетов.

Сторонние гемы могут понадобиться разработчикам, желающим использовать клиентские фреймворки вместе с Ханами.
Предположим, мы хотим создать гем `hanami-emberjs`.

```shell
% tree .
# ...
├── lib
│   └── hanami
│       ├── emberjs
│       │   ├── dist
│       │   │   ├── ember.js
│       │   │   └── ember.min.js
│       │   └── version.rb
│       └── emberjs.rb
├── hanami-emberjs.gemspec
# ...
```

Мы выбрали **произвольную** папку и сохранили в нее **только** файлы, которые должны быть доступны извне.
Затем мы просто добавили ее в `Hanami::Assets.sources`.

```ruby
# lib/hanami/emberjs.rb
require 'hanami/assets'

module Hanami
  module Emberjs
    require 'hanami/emberjs/version'
  end
end

Hanami::Assets.sources << __dir__ + '/emberjs/source'
```

Когда приложение выполнит строку `require 'hanami/emberjs'`, эта папка будет добавлена в список потенциальных источников файлов и при запросе Ханами сможет обслуживать файлы из нее.

```erb
<%= javascript 'ember' %>
```

Для включения файла `ember.js` мы можем использовать [хэлпер](/guides/helpers/assets) `javascript`.
