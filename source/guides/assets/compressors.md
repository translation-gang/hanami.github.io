---
title: Руководство - Сжатие ассетов
---

# Ассеты

## Сжатие

Сжатие ассетов, или минификация — это процесс сокращения размеров файлов, благодаря которому браузер будет быстрее загружать их.
Обычно оно применяется к таблицам стилей и сценариям JavaScript.

За включение этой функции отвечают следующие строки в `apps/web/application.rb`:

```ruby
# apps/web/application.rb
module Web
  class Application < Hanami::Application
    configure do
      assets do
        javascript_compressor :builtin
        stylesheet_compressor :builtin

        # ...
      end
    end
  end
end
```

Чтобы отключить ее, достаточно просто закомментировать одну или каждую из этих строк.

### JavaScript

Мы поддерживаем следующие движки для сжатия:

  * `:builtin` - использующий не самые эффективные алгоритмы, но зато написанный на **чистом Руби** и **не требует разрешения внешних зависимостей**.
  * `:yui` - использующий [Yahoo! YUI Compressor](http://yui.github.io/yuicompressor). Требует установки гема [`yui-compressor`](https://rubygems.org/gems/yui-compressor) и Java 1.4+.
  * `:uglifier` - использующий [UglifyJS2](http://lisperator.net/uglifyjs). Требует установки гема [uglifier](https://rubygems.org/gems/uglifier) и Node.js.
  * `:closure` - использующий [Google Closure Compiler](https://developers.google.com/closure/compiler). Требует установки гема [`closure-compiler`](https://rubygems.org/gems/closure-compiler) и Java.

<p class="warning">
  Для использования движков сжатия <code>:yui</code>, <code>:uglifier</code> и <code>:closure</code>, необходимо добавить соответствующие гемы в <code>Gemfile</code> проекта.
</p>

### Таблицы стилей

Мы поддерживаем следующие движки для сжатия:

  * `:builtin` - использующий не самые эффективные алгоритмы, но зато написанный на **чистом Руби** и **не требует разрешения внешних зависимостей**.
  * `:yui` - использующий [Yahoo! YUI Compressor](http://yui.github.io/yuicompressor). Требует установки гема [`yui-compressor`](https://rubygems.org/gems/yui-compressor) и Java 1.4+.
  * `:sass` - использующий [Sass](http://sass-lang.com). Требует установки гема [sass](https://rubygems.org/gems/sass).

<p class="warning">
  Для использования движков сжатия <code>:yui</code> и <code>:sass</code>, необходимо добавить соответствующие гемы в <code>Gemfile</code> проекта.
</p>

### Пользовательские движки

Мы можем использовать собственный движок для **JS и CSS**.
Соответствующий объект **должен** иметь метод `#compress(filename)` и возвращать `String` со сжатым содержимым.

```ruby
class MyCustomJavascriptCompressor
  def compress(filename)
    # ...
  end
end
```

А затем этот объект нужно передать в конфигурацию:

```ruby
# apps/web/application.rb
module Web
  class Application < Hanami::Application
    configure do
      assets do
        javascript_compressor MyCustomJavascriptCompressor.new
        stylesheet_compressor :builtin

        # ...
      end
    end
  end
end
```
