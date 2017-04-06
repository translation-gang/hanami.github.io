---
title: Руководство - Хэлперы: Хэлперы форм
---

## Хэлперы форм

В Ханами встроен мощный API для описания HTML5 форм при помощи Руби. Он используется в представлениях и шаблонах. В его состав входят:

   * Поддержка полноценной разметки без необходимости конкатенации;
   * Автоматическое закрытие тэгов HTML5;
   * Локальные переменные представлений;
   * Переназначение методов (`PUT`/`PATCH`/`DELETE` HTTP глаголы, не воспринимаемые браузерами);
   * Автоматическая генерация HTML атрибутов для полей ввода: `id`, `name`, `value`;
   * Распознавание значений параметров запроса и/или используемых сущностей для автоматической подстановки в атрибут `value`;
   * Автоматический выбор состояния переключателей; 
   * Защита от межсайтовой подделки запросов(CSRF);
   * Неограниченная вложенность полей;
   * Независимость от ORM.

## Технические особенности

Многие гемы со схожим назначением имеют аналогичный синтаксис, но в Rails или Padrino они используются несколько иначе.

Синтаксис этих фреймворков может выглядеть подобным образом:

```erb
<%= form_for :book do |f| %>
  <div>
    <%= f.text_field :title %>
  </div>
<% end %>
```

Но такой код **не является корректным с точки зрения синтаксиса ERB**.
Чтобы заставить его работать, эти фреймворки используют собственные ERB обработчики или используют сторонние гемы в случае других шаблонизаторов.

### Независимость от шаблонизаторов

Ханами стремится обеспечить поддержку любых шаблонизаторов. Она достигается упрощением интерфейса. Хэлперы используют только то, что уже встроено в шаблонизатор.
Это означает, что мы можем без дополнительных усилий использовать Slim, HAML или ERB.

### Единственный блок для вывода

Соблюдение вышеописанного принципа достигается при помощи небольшого компромисса. Необходимо встраивать форму внутри единственного блока:

```erb
<%=
  form_for :book, routes.books_path do
    text_field :title

    submit 'Create'
  end
%>
```

Будет преобразовано в:

```html
<form action="/books" id="book-form" method="POST">
  <input type="hidden" name="_csrf_token" value="0a800d6a8fc3c24e7eca319186beb287689a91c2a719f1cbb411f721cacd79d4">
  <input type="text" name="book[title]" id="book-id" value="">
  <button type="submit">Create</button>
</form>
```

### Методы представлений

В качестве альтернативы можно определить необходимые методы внутри представлений:

```ruby
module Web::Views::Books
  class New
    include Web::View

    def form
      form_for :book, routes.books_path do
        text_field :title

        submit 'Create'
      end
    end
  end
end
```

```erb
<%= form %>
```

### Поддерживаемые методы

* [check_box](http://www.rubydoc.info/gems/hanami-helpers/Hanami/Helpers/FormHelper/FormBuilder#check_box-instance_method)
* [color_field](http://www.rubydoc.info/gems/hanami-helpers/Hanami/Helpers/FormHelper/FormBuilder#color_field-instance_method)
* [datalist](http://www.rubydoc.info/gems/hanami-helpers/Hanami/Helpers/FormHelper/FormBuilder#datalist-instance_method)
* [date_field](http://www.rubydoc.info/gems/hanami-helpers/Hanami/Helpers/FormHelper/FormBuilder#date_field-instance_method)
* [datetime_field](http://www.rubydoc.info/gems/hanami-helpers/Hanami/Helpers/FormHelper/FormBuilder#datetime_field-instance_method)
* [datetime_local_field](http://www.rubydoc.info/gems/hanami-helpers/Hanami/Helpers/FormHelper/FormBuilder#datetime_local_field-instance_method)
* [email_field](http://www.rubydoc.info/gems/hanami-helpers/Hanami/Helpers/FormHelper/FormBuilder#email_field-instance_method)
* [fields_for](http://www.rubydoc.info/gems/hanami-helpers/Hanami/Helpers/FormHelper/FormBuilder#fields_for-instance_method)
* [file_field](http://www.rubydoc.info/gems/hanami-helpers/Hanami/Helpers/FormHelper/FormBuilder#file_field-instance_method)
* [form_for](http://www.rubydoc.info/gems/hanami-helpers/Hanami/Helpers/FormHelper#form_for-instance_method)
* [hidden_field](http://www.rubydoc.info/gems/hanami-helpers/Hanami/Helpers/FormHelper/FormBuilder#hidden_field-instance_method)
* [label](http://www.rubydoc.info/gems/hanami-helpers/Hanami/Helpers/FormHelper/FormBuilder#label-instance_method)
* [number_field](http://www.rubydoc.info/gems/hanami-helpers/Hanami/Helpers/FormHelper/FormBuilder#number_field-instance_method)
* [password_field](http://www.rubydoc.info/gems/hanami-helpers/Hanami/Helpers/FormHelper/FormBuilder#password_field-instance_method)
* [radio_button](http://www.rubydoc.info/gems/hanami-helpers/Hanami/Helpers/FormHelper/FormBuilder#radio_button-instance_method)
* [select](http://www.rubydoc.info/gems/hanami-helpers/Hanami/Helpers/FormHelper/FormBuilder#select-instance_method)
* [submit](http://www.rubydoc.info/gems/hanami-helpers/Hanami/Helpers/FormHelper/FormBuilder#submit-instance_method)
* [text_area](http://www.rubydoc.info/gems/hanami-helpers/Hanami/Helpers/FormHelper/FormBuilder#text_area-instance_method)
* [text_field](http://www.rubydoc.info/gems/hanami-helpers/Hanami/Helpers/FormHelper/FormBuilder#text_field-instance_method)

## Примеры

### Общий случай использования

Наш API очень чист и выразителен. **Он не требует конкатенировать** возвращаемые значения блоков(`submit`) и предшествующих строк в шаблоне(`div`).

```erb
<%=
  form_for :book, routes.books_path, class: 'form-horizontal' do
    div do
      label      :title
      text_field :title, class: 'form-control'
    end

    submit 'Create'
  end
%>
```

```html
<form action="/books" id="book-form" method="POST" class="form-horizontal">
  <input type="hidden" name="_csrf_token" value="1825a0a7ea92bbe3fd60cc8b6a0ea00ce3c52030afbf4037370d937bc5248acb">
  <div>
    <label for="book-title">Title</label>
    <input type="text" name="book[title]" id="book-title" value="" class="form-control">
  </div>

  <button type="submit">Create</button>
</form>
```

### Переназначение методов 

Браузеры не могут использовать HTTP методы помимо `GET` и `POST`. Ханами же придерживается соглашения REST, которое выходит за рамки этих двух методов. Поэтому когда указан метод при помощи`:method`, в форму будет встроено скрытое поле `_method`, которое будет воспринято приложением.

```erb
<%=
  form_for :book, routes.book_path(book.id), method: :put do
    text_field :title

    submit 'Update'
  end
%>
```

```html
<form action="/books/23" id="book-form" method="POST">
  <input type="hidden" name="_method" value="PUT">
  <input type="hidden" name="_csrf_token" value="5f1029dd15981648a0882ec52028208410afeaeffbca8f88975ef199e2988ce7">
  <input type="text" name="book[title]" id="book-title" value="Test Driven Development">

  <button type="submit">Update</button>
</form>
```

### Защита от межсайтовой подделки запросов(CSRF)

Межсайтовая подделка запросов(CSRF) является одной из самых распространенных веб-атак. Ханами предлагает использовать механизм защиты основанный на _синхронизации по токену_.

Когда мы включаем сессии, у каждого пользователя появляется случайно сгенерированный токен.
Формы же создаются со специальным скрытым полем(`_csrf_token`), которое передает его.

При отправке формы Ханами сравнит его значение со значением сессии. Если они совпадут, то запрос будет обработан, а иначе произойдет сброс сессии.

Разработчики могут модифицировать этот механизм.

### Вложенные поля

```erb
<%=
  form_for :delivery, routes.deliveries_path do
    text_field :customer_name

    fields_for :address do
      text_field :city
    end

    submit 'Create'
  end
%>
```

```html
<form action="/deliveries" id="delivery-form" method="POST">
  <input type="hidden" name="_csrf_token" value="4800d585b3a802682ae92cb72eed1cdd2894da106fb4e9e25f8a262b862c52ce">
  <input type="text" name="delivery[customer_name]" id="delivery-customer-name" value="">
  <input type="text" name="delivery[address][city]" id="delivery-address-city" value="">

  <button type="submit">Create</button>
</form>
```

## Автозаполнение полей

Поля формы могут быть **автоматически заполнены**. Ханами заполнит те поля формы, для которых будут найдены значения в конструкторе формы или в параметрах запроса.

#### Пример

Представим, что нам необходимо обновить данные для доставки `delivery`. У нас есть два объекта: доставка(`delivery`) и заказчик(`customer`), которые являются простыми объектами(не связанными с ORM) и могут отвечать на вызовы следующих методов:

```ruby
delivery.id   # => 1
delivery.code # => 123

customer.name # => "Luca"

customer.address.class # => Address
customer.address.city  # => "Rome"
```

Составим форму:

```erb
<%=
  form_for :delivery, routes.delivery_path(id: delivery.id), method: :patch, values: {delivery: delivery, customer: customer} do
    text_field :code

    fields_for :customer do
      text_field :name

      fields_for :address do
        text_field :city
      end
    end

    submit 'Update'
  end
%>
```

```html
<form action="/deliveries/1" id="delivery-form" method="POST">
  <input type="hidden" name="_method" value="PATCH">
  <input type="hidden" name="_csrf_token" value="4800d585b3a802682ae92cb72eed1cdd2894da106fb4e9e25f8a262b862c52ce">

  <input type="text" name="delivery[code]" id="delivery-code" value="123">

  <input type="text" name="delivery[customer][name]" id="delivery-customer-name" value="Luca">
  <input type="text" name="delivery[customer][address][city]" id="delivery-customer-address-city" value="Rome">

  <button type="submit">Update</button>
</form>
```

Обратите внимание на параметр `:values`, который мы передали в метод `#form_for`. Он задаст поле формы `name`, которое мы хотим сделать заполненным. Экземпляр `delivery[code]` будет передан в `delivery.code` (`123`), а `delivery[customer][address][city]` в `customer.address.city` (`"Rome"`) и т.д..

### Чтение из параметров

**Параметры автоматически передаются в хэлперы форм** чтобы попытаться заполнить поля. Если значение присутствует и в параметрах, и в (`:values`), то будет выбрано значение из параметров. Причина такого решения проста: параметры запроса часто содержат значения форм, заполненных с ошибками.

#### Пример

Возьмем форму из предыдущего примера. Допустим, пользователь ввел `"foo"` в поле для кода заказа. Это значение не удовлетворяет логике нашего приложения. 
Тогда мы предложим пользователю исправить значения в форме указав на ошибку. Теперь форма будет передана со значениями, уже заполненными пользователем. Например, `params.get('delivery.code')` будет содержать `"foo"`.

Вот как будет выглядеть эта форма:

```html
<form action="/deliveries/1" id="delivery-form" method="POST">
  <input type="hidden" name="_method" value="PATCH">
  <input type="hidden" name="_csrf_token" value="4800d585b3a802682ae92cb72eed1cdd2894da106fb4e9e25f8a262b862c52ce">

  <input type="text" name="delivery[code]" id="delivery-code" value="foo">

  <input type="text" name="delivery[customer][name]" id="delivery-customer-name" value="Luca">
  <input type="text" name="delivery[customer][address][city]" id="delivery-customer-address-city" value="Rome">

  <button type="submit">Update</button>
</form>
```
