---
title: Руководство - Модели: Сущности
---

# Сущности

Сущности — это часть модели, описывающая бизнес-логику приложения.
Эрик Эванс подробно описал их в книге "Проблемно-ориентированное проектирование".

Сущности составляют ядро приложения. Они представлены небольшими объектами и реализуют самые важные целевые функции.

Они спроектированы согласно принципу единственной ответственности(single responsibility) и не предназначены для проведения валидации данных и обеспечения их хранения.

Такая простота позволяет разработчику сосредоточиться на поведении объекта, то есть сообщениях, на которые он отвечает.
Именно так, как это предписывают принципы объектно-ориентированного программирования.

## Структура сущности

Сущности содержат структуру своих атрибутов, их имена и типы.
Структура выполняет роль фильтра для данных. Если в дальнейшем поступающие данные не будут соответствовать этой схеме, то будет вызвано исключение.

### Автоматическая разметка

При использовании реляционной базы данных структура формируется автоматически исходя из структуры таблицы.

Представим, что у нас есть следующая таблица `books`:

```sql
CREATE TABLE books (
    id integer NOT NULL,
    title text,
    created_at timestamp without time zone,
    updated_at timestamp without time zone
);
```

А это соответствующая сущность `Book`:

```ruby
# lib/bookshelf/entities/book.rb
class Book < Hanami::Entity
end
```

---

Попробуем создать ее экземпляр:

```ruby
book = Book.new(title: "Hanami")

book.title      # => "Hanami"
book.created_at # => nil
```

Атрибут `created_at` равен `nil` из-за того, что мы не указали его явно в конструкторе `book`.

---

Конструктор проигнорирует неизвестные атрибуты:

```ruby
book = Book.new(unknown: "value")

book.unknown # => NoMethodError
book.foo     # => NoMethodError
```

Будет вызвано исключение `NoMethodError` для `unknown` и `foo`. Они не были описаны в структуре сущности.

---

Конструктор попытается привести данные к ожидаемому типу:

```ruby
book = Book.new(created_at: "Sun, 13 Nov 2016 09:41:09 GMT")

book.created_at       # => 2016-11-13 09:41:09 UTC
book.created_at.class # => Time
```

Он предпримет несколько попыток пока не приведет данные к типу, указанному в структуре.

---

Если все попытки провалятся, то будет вызвано исключение:

```ruby
Book.new(created_at: "foo") # => ArgumentError
```

В комбинации с [валидациями базы данных](/guides/migrations/create-table#constraints) и валидациями на других уровнях, мы можем поддерживать **высокий уровень целостности данных** в наших проектах.

### Собственная структура

Мы можем поднять поддержание целостности данных на следующую ступень, если дополним внутреннюю структуру.

<p class="notice">
  Пользовательская структура <strong>опционально добавляется</strong> в случае реляционных баз данных, но обязательна для хранилищ, в которых нет структурированных таблиц.
</p>

```ruby
# lib/bookshelf/entities/user.rb
class User < Hanami::Entity
  EMAIL_FORMAT = /\@/

  attributes do
    attribute :id,         Types::Int
    attribute :name,       Types::String
    attribute :email,      Types::String.constrained(format: EMAIL_FORMAT)
    attribute :age,        Types::Int.constrained(gt: 18)
    attribute :codes,      Types::Collection(Types::Coercible::Int)
    attribute :comments,   Types::Collection(Comment)
    attribute :created_at, Types::Time
    attribute :updated_at, Types::Time
  end
end
```

Создадим экземпляр такой сущности:

```ruby
user = User.new(name: "Luca", age: 34, email: "test@hanami.test")

user.name     # => "Luca"
user.age      # => 34
user.email    # => "luca@hanami.test"
user.codes    # => nil
user.comments # => nil
```

---

Значения все еще могут быть приведены к ожидаемому типу:

```ruby
user = User.new(codes: ["123", "456"])
user.codes # => [123, 456]
```

Частью сущностей могут быть и другие сущности:

```ruby
user = User.new(comments: [Comment.new(text: "cool")])
user.comments
  # => [#<Comment:0x007f966be20c58 @attributes={:text=>"cool"}>]
```

Или их можно передать в виде хэша:

```ruby
user = User.new(comments: [{text: "cool"}])
user.comments
  # => [#<Comment:0x007f966b689e40 @attributes={:text=>"cool"}>]
```

---

**Целостность данных** будет поддерживаться путем вызова исключений:

```ruby
User.new(email: "foo")     # => TypeError: "foo" (String) has invalid type for :email
User.new(comments: [:foo]) # => TypeError: :foo must be coercible into Comment
```

---

<p class="warning">
  Пользовательская схема данных <strong>добавляет точности</strong> автоматически созданной схеме.
  Если мы хотим ее использовать, то сначала необходимо вручную добавить структуру таблицы базы данных.
</p>

---

Узнать больше о типах данных можно в [соответствующем разделе](/guides/models/data-types).
