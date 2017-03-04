---
title: Руководство - Шаблоны мэйлеров
---

# Шаблоны

Шаблоном называется файл определенного формата, описывающий содержимое будущего электронного письма.
Например, файл `welcome.html.erb` описывает HTML составляющую письма, а `welcome.txt.erb` текстовую.

Их обработка будет выполняться _шаблонизатором_ в [контексте](/guides/mailers/basic-usage) мэйлера.

## Именование

По общепринятому соглашению имя мэйлера определяет имя соответствующего шаблона.
Так для мэйлера `Mailers::ForgotPassword` будет использован шаблон начинающийся с `forgot_password`.

Оставшаяся часть имени будет состоять из нескольких расширений файла.
Первое из них определит **_формат_**, а последующие – **_шаблонизаторы_**.

**Ханами использует для электронных писем только форматы `html` и `txt`.**

<p class="convention">
Для мэйлера с именем <code>Mailers::ForgotPassword</code> должен существовать хотя бы один шаблон с именем формата <code>forgot_password.[формат].[шаблонизатор]</code> в папке почтовых шаблонов.
</p>

### Нестандартные шаблоны

Если мы хотим воспользоваться другим шаблоном, то можем вызвать метод `template`.

```ruby
# lib/bookshelf/mailers/forgot_password.rb
class Mailers::ForgotPassword
  include Hanami::Mailer
  template 'send_password'
end
```

Тогда мэйлер будет искать шаблон `lib/bookshelf/mailers/templates/send_password.*`.

## Шаблонизаторы

Ханами определяет шаблонизатор исходя из последнего расширения файла шаблона. Так для файла `welcome.html.erb` будет использован ERb.

В качестве стандартного шаблонизатора используется [ERb](http://ru.wikipedia.org/wiki/ERuby), но Ханами также поддерживает огромное число и других шаблонизаторов.

Ниже представлен их список.
Они перечислены в порядке **предпочтения для отдельного формата**.
Например, если установлен [ERubis](http://www.kuwata-lab.com/erubis/), то именно он будет использован для шаблонов`.erb`.

<table class="table table-bordered table-striped">
  <tr>
    <th>Шаблонизатор</th>
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

Чтобы начать использовать любой из них, достаточно включить нужный гем в сборку и указать соответствующее расширение файла.

```haml
# lib/bookshelf/mailers/templates/welcome.html.haml
%h1 Welcome
```

## Папка шаблонов

По умолчанию шаблоны располагаются в `mailers/templates`, находящейся внутри папки приложения `lib/bookshelf`, где `bookshelf` это имя приложения.
Для переопределения каталога необходимо явно указать его в Hanami::Mailer.

```ruby
# lib/bookshelf.rb
# ...

Hanami::Mailer.configure do
  # ...
  root 'path/to/templates'
end.load!
```

В этом случае приложение будет искать шаблоны в `path/to/templates`.
