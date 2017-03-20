---
title: Руководство - Представления: Шаблоны 
---

# Шаблоны

Шаблоны — это файлы, описывающие структуру тела ответа на HTTP запрос.
При помощи представлений формируется контекст, данные для вывода. Затем _шаблонизатор_ применяя эти данные к шаблону составляет тело ответа.

## Именование

Для простоты имя представления соотносится с именем файла шаблона.
Полное имя класса представления соответствует пути к шаблону: представлению `Dashboard::Index` соответствует шаблон `dashboard/index`.

Остальная часть пути зависит от расширений файла.
Первое расширение описывает **_формат конечного файла_**, а второе **_используемый шаблонизатор_**.

<p class="convention">

Представлению с именем <code>Web::Views::Dashboard::Index</code> должен соответствовать шаблон <code>dashboard/index.[расширение конечного файла].[шаблонизатор]</code> в папке шаблонов.
</p>

## Вложенные шаблоны

Сборка одного шаблона из нескольких частей доступна при помощи метода `render`, если передать ему `partial` и путь к желаемому шаблону.

```
# Вкладываемый шаблон:
#   templates/shared/_sidebar.html.erb
#
# Шаблон-родитель:
#   templates/application.html.erb
#
<%= render partial: 'shared/sidebar' %>
```


Вставка одного шаблон в другой происходит при помощи метода `render` с аргументом `template`:

```
# Вкладываемый шаблон:
#   templates/articles/index.html.erb
#
# Шаблон-родитель:
#   templates/application.html.erb
#
<%= render template: 'articles/index' %>
```

### Нестандартные шаблоны

Имя и путь к шаблону можно задать явно при помощи `template`.

```ruby
# apps/web/views/dashboard/index.rb
module Web::Views::Dashboard
  class Index
    include Web::View
    template 'home/index'
  end
end
```

Представление будет искать шаблон `apps/web/templates/home/index.*`.

## Шаблонизаторы

Последнее расширение шаблона определяет, какой шаблонизатор Ханами будет использовать(в случае `index.html.erb` будет использоваться ERb).
По умолчанию генерируются шаблоны [ERb](http://ru.wikipedia.org/wiki/ERuby), но Ханами поддерживает бесчисленное множество других шаблонизаторов "из коробки".

Вот их список.
Они указаны в порядке **возрастания приоритета** для своего расширения.
Например, если [ERubis](http://www.kuwata-lab.com/erubis/) доступен, то он будет использоваться вместо ERb для файлов `.erb`.

<table class="table table-bordered table-striped">
  <tr>
    <th>Шабонизатор</th>
    <th>Расширения</th>
  </tr>
  <tr>
    <td>Erubis</td>
    <td>erb, rhtml, erubis</td>
  </tr>
  <tr>
    <td>ERb</td>
    <td>erb, rhtml</td>
  </tr>
  <tr>
    <td>Redcarpet</td>
    <td>markdown, mkd, md</td>
  </tr>
  <tr>
    <td>RDiscount</td>
    <td>markdown, mkd, md</td>
  </tr>
  <tr>
    <td>Kramdown</td>
    <td>markdown, mkd, md</td>
  </tr>
  <tr>
    <td>Maruku</td>
    <td>markdown, mkd, md</td>
  </tr>
  <tr>
    <td>BlueCloth</td>
    <td>markdown, mkd, md</td>
  </tr>
  <tr>
    <td>Asciidoctor</td>
    <td>ad, adoc, asciidoc</td>
  </tr>
  <tr>
    <td>Builder</td>
    <td>builder</td>
  </tr>
  <tr>
    <td>CSV</td>
    <td>rcsv</td>
  </tr>
  <tr>
    <td>CoffeeScript</td>
    <td>coffee</td>
  </tr>
  <tr>
    <td>WikiCloth</td>
    <td>wiki, mediawiki, mw</td>
  </tr>
  <tr>
    <td>Creole</td>
    <td>wiki, creole</td>
  </tr>
  <tr>
    <td>Etanni</td>
    <td>etn, etanni</td>
  </tr>
  <tr>
    <td>Haml</td>
    <td>haml</td>
  </tr>
  <tr>
    <td>Less</td>
    <td>less</td>
  </tr>
  <tr>
    <td>Liquid</td>
    <td>liquid</td>
  </tr>
  <tr>
    <td>Markaby</td>
    <td>mab</td>
  </tr>
  <tr>
    <td>Nokogiri</td>
    <td>nokogiri</td>
  </tr>
  <tr>
    <td>Plain</td>
    <td>html</td>
  </tr>
  <tr>
    <td>RDoc</td>
    <td>rdoc</td>
  </tr>
  <tr>
    <td>Radius</td>
    <td>radius</td>
  </tr>
  <tr>
    <td>RedCloth</td>
    <td>textile</td>
  </tr>
  <tr>
    <td>Sass</td>
    <td>sass</td>
  </tr>
  <tr>
    <td>Scss</td>
    <td>scss</td>
  </tr>
  <tr>
    <td>Slim</td>
    <td>slim</td>
  </tr>
  <tr>
    <td>String</td>
    <td>str</td>
  </tr>
  <tr>
    <td>Yajl</td>
    <td>yajl</td>
  </tr>
</table>

Чтобы использовать несколько шаблонизаторов необходимо просто добавить их в проект и указать соответствующие расширения файлов.

```haml
# app/web/templates/dashboard/index.html.haml
%h1 Dashboard
```

## Каталог

Шаблоны по умолчанию хранятся в папке `templates` внутри своего приложения `apps/web`.
Для смены этой папки необходимо добавить следующую строку в файл конфигурации.

```ruby
# apps/web/application.rb
module Web
  class Application < Hanami::Application
    configure do
      # ...
      templates 'path/to/templates'
    end
  end
end
```

Теперь приложение будет искать шаблоны в папке `apps/web/path/to/templates`.
