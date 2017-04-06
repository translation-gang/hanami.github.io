---
title: Руководство - Препроцессоры для ассетов
---

# Ассеты

## Препроцессоры

Ханами позволяет использовать препроцессоры для ассетов и загружать их в `public/assets`.

Допустим, у нас есть файл `application.css.scss` в папке `apps/web/assets/stylesheets` и `reset.css` в
`apps/web/vendor/stylesheets`.

**Структура расширений имеет значение.**
Первое определяет тип ассетов, оно обязательное. В данном случае это `.css`, то есть таблицы стилей.
Следующее добавляется опционально и оно определит используемый препроцессор: `.scss`, то есть препроцессор Sass.

<p class="convention">
  Если дан файл <code>application.css.scss</code>, то по его последнему расширению (<code>.scss</code>) будет определен препроцессор для обработки.
</p>

<p class="notice">
  Использование препроцессоров не обязательно. Приложение может работать с чистым JavaScript или таблицами стилей. В вышеописанном примере будет использован только один препроцессор. Их может быть как больше, а может не быть вовсе.
</p>

```ruby
# apps/web/application.rb
require 'sass'

module Web
  class Application < Hanami::Application
    configure do
      # ...

      assets do
        sources << [
          # apps/web/assets используется по умолчанию
          'vendor/assets' # app/web/vendor/assets
        ]
      end
    end
  end
end
```

В шаблоне мы напишем:

```erb
<%= stylesheet 'reset', 'application' %>
```

А когда будет запрошена страница, препроцессор обработает файл стилей и скопирует его в `public/assets`.

```shell
% tree public
public/
└── assets
    ├── application.css
    └── reset.css
```

Препроцессор будет обрабатывать и копировать файлы только если включен [_Режим Прекомпиляции_](/guides/assets/overview).

<p class="convention">
  Препроцессоры по умолчанию включены только в окружении <em>разработки</em> и <em>тестирования</em>.
</p>

В целях повышения производительности эта функция отключена в режиме _эксплуатации_, в котором следует заранее [прекомпилировать](/guides/command-line/assets) ассеты.

### Движки препроцессоров

Ханами использует [Tilt](https://github.com/rtomayko/tilt) для обеспечения поддержки наиболее востребованных препроцессоров: [Sass](http://sass-lang.com/) (включая `sassc-ruby`), [Less](http://lesscss.org/), ES6, [JSX](https://jsx.github.io/), [CoffeScript](http://coffeescript.org), [Opal](http://opalrb.org), [Handlebars](http://handlebarsjs.com), [JBuilder](https://github.com/rails/jbuilder).

Перед тем как использовать один или несколько из них, убедитесь, что соответсвующий гем включен в ваш проект через `Gemfile`.

```ruby
# Gemfile
# ...
gem 'sass'
```

<p class="notice">
  Некоторые препроцессоры требуют установки Node.js. Внимательно изучайте документацию препроцессоров.
</p>

#### EcmaScript 6

Мы настоятельно рекомендуем использовать [EcmaScript 6](http://es6-features.org/) в ваших проектах, это самый новый стандарт JavaScript.
Он еще не полностью [поддерживается](https://kangax.github.io/compat-table/es6/) в большинстве браузеров, но это скоро изменится.

А пока нам придется транспилировать код на ES6 в то, что будет работать в большинстве браузеров, то есть ES5.
Для этого потребуется [Babel](https://babeljs.io).

<p class="notice">
  Babel не работает без Node.js.
</p>
