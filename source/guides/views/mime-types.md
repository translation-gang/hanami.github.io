---
title: Руководство - Представления: MIME типы
---

# MIME типы

Представления могут работать с несколькими MIME типами. Перед тем как углубиться в эту тему, убедитесь, что вы знакомы с [обработкой MIME типов в экшенах](/guides/actions/mime-types).

Важно понимать как зависит имя шаблона от _формата_.
Для каждого MIME типа у Rack(а значит и у Ханами) есть ассоциированный с ним _формат_.
В случае XML `application/xml` соответствует `:xml`, а HTML `text/html` станет `:html`.

<p class="convention">
Формат определяется по первому расширению файла шаблона. Например, <code>dashboard/index.html.*</code>.
</p>

## Шаблонизация по умолчанию

Если экшн (`Web::Controllers::Dashboard::Index`) примет запрос JSON и для него существует шаблон (`apps/web/templates/dashboard/index.json.erb`), то представление использует этот шаблон.

```erb
# apps/web/templates/dashboard/index.json.erb
{"foo":"bar"}
```

```shell
% curl -H "Accept: application/json" http://localhost:2300/dashboard
{"foo":"bar"}
```

При этом мы все еще можем запросить ответ в формате HTML.

```erb
# apps/web/templates/dashboard/index.html.erb
<h1>Dashboard</h1>
```

```shell
% curl -H "Accept: text/html" http://localhost:2300/dashboard
<h1>Dashboard</h1>
```

В случае запроса неподдерживаемого MIME типа приложение сообщит об ошибке.

```shell
% curl -H "Accept: application/xml" http://localhost:2300/dashboard
Hanami::View::MissingTemplateError: Can't find template "dashboard/index" for "xml" format.
```

## Представление нестандартного формата

Вышеописанный сценарий хорошо работает когда формат поддерживается по умолчанию.
Но что если мы хотим [обработать собственный формат](/guides/views/basic-usage) или реализовать дополнительную логику в представлении?

Мы можем создать класс наследующий стандартному представлению и позволить ему обработать этот формат.

```ruby
# apps/web/views/dashboard/json_index.rb
require_relative './index'

module Web::Views::Dashboard
  class JsonIndex < Index
    format :json

    def render
      raw JSON.generate({foo: 'bar'})
    end
  end
end
```

Запрос JSON на `/dashboard` будет обработан `JsonIndex`.

<p class="notice">
Не существует соглашения касательно имен таких классов. Важно обращать внимание на <code>format :json</code>.
</p>

Пример выше показывает как использовать нестандартные методы обработки и заменить шаблоны на сериализацию JSON.
