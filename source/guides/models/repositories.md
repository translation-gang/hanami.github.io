---
title: Руководство - Модели: Репозитории
---

# Репозитории

Репозитории — это объекты посредники между сущностями и хранилищем данных.
Они предлагают стандартизированный API для запросов и команд базе данных.

Репозитории **не зависят от конкретной базы данных**. Все действия делегируются соответствующим адаптерам.

У такой архитектуры есть ряд достоинств:

  * Приложения зависят от надежного API, а не от низкоуровневых деталей реализации(принцип инверсии зависимостей);

  * Если меняется база данных, то нужно изменить только адаптер, а не само приложение;

  * Во время разработки можно отложить выбор базы данных;

  * Логика хранения данных отделена от логики взаимодействий с ними;

  * Несколько хранилищ данных могут легко сосуществовать в рамках одного приложения;

<p class="warning">
  На текущий момент Ханами поддерживает только реляционные базы данных
</p>

## Интерфейс

Когда класс наследует от `Hanami::Repository`, он получает следующий интерфейс:

  * `#create(data)` – создание записи с переданными данными, возвращает сущность;
  * `#update(id, data)` – изменение записи с данным id, возвращает сущность;
  * `#delete(id)` – удаление записи с переданным id;
  * `#all` – получение всех записей из коллекции;
  * `#find(id)` – получение всех записей из коллекции с переданным id;
  * `#first` – получение первой записи из коллекции;
  * `#last`  – получение последней записи из коллекции;
  * `#clear` – удаление всех записей из коллекции.

**Коллекция – набор записей с общей структурой.**
Это может быть таблица в случае реляционных баз данных или коллекция MongoDB.

```ruby
repository = BookRepository.new

book = repository.create(title: "Hanami")
  # => #<Book:0x007f95cbd8b7c0 @attributes={:id=>1, :title=>"Hanami", :created_at=>2016-11-13 16:02:37 UTC, :updated_at=>2016-11-13 16:02:37 UTC}>

book = repository.find(book.id)
  # => #<Book:0x007f95cbd5a030 @attributes={:id=>1, :title=>"Hanami", :created_at=>2016-11-13 16:02:37 UTC, :updated_at=>2016-11-13 16:02:37 UTC}>

book = repository.update(book.id, title: "Hanami Book")
  # => #<Book:0x007f95cb243408 @attributes={:id=>1, :title=>"Hanami Book", :created_at=>2016-11-13 16:02:37 UTC, :updated_at=>2016-11-13 16:03:34 UTC}>

repository.delete(book.id)

repository.find(book.id)
  # => nil
```

## Запросы

**Все запросы являются закрытыми**.
Такое решение вынуждает разработчиков работать с API, а не с деталями реализации хранилища.

Рассмотрим следующий код:

```ruby
BookRepository.new.where(author_id: 23).order(:published_at).limit(8)
```

Он **плох** сразу по ряду причин:

  * Вызывающий объект знает слишком много о внутренних механизмах хранилища;

  * Вызывающий объект работает сразу с нескольими уровнями абстракции;

  * Этот код не выразителен: он не отражает цель вызова, а содержит целую цепочку методов;

  * Вызывающий объект становится трудее изолировать для тестов;

  * Если придется менять хранилище, то и этот код придется изменить.

Есть более подходящий способ:

```ruby
# lib/bookshelf/repositories/book_repository.rb
class BookRepository < Hanami::Repository
  def most_recent_by_author(author, limit: 8)
    books
      .where(author_id: author.id)
      .order(:published_at)
      .limit(limit)
  end
end
```

Это **гораздо лучше**, потому что:

  * Вызывающий объект больше не будет знать, как репозиторий получает эти сущности;

  * Вызывающий объект останется на своем уровне абстракции, на котором ничего неизвестно о записях и вся работа происходит только с использованием сущностей;

  * Имя этого метода отражает его назначание;

  * Вызываемый объект легко можно будет протестировать отдельно от контекста, сделав заглушку этого метода;

  * Если придется менять хранилище, то код вызывающего объекта останется тем же.

## Временные метки

Бывает чрезвычайно полезно отслеживать даты создания и изменения записей, особенно когда приложение уже введено в эксплуатацию.

Во время создания новой таблицы мы можем задать следующие столбцы. Тогда репозитории будут автоматически заполнять их значения.

```ruby
Hanami::Model.migration do
  up do
    create_table :books do
      # ...
      column :created_at, DateTime
      column :updated_at, DateTime
    end
  end
end
```

```ruby
repository = BookRepository.new

book = repository.create(title: "Hanami")

book.created_at # => 2016-11-14 08:20:44 UTC
book.updated_at # => 2016-11-14 08:20:44 UTC

book = repository.update(book.id, title: "Hanami Book")

book.created_at # => 2016-11-14 08:20:44 UTC
book.updated_at # => 2016-11-14 08:22:40 UTC
```

<p class="convention">
  Если при создании таблицы базы данных указаны столбцы <code>created_at</code> и <code>updated_at</code>, то репозитории автоматически будут устанавливать в них временные метки.
</p>

<p class="notice">
  Временные метки устанавливаются с учетом UTC.
</p>

## Унаследованные базы данных

По умолчанию репозиторий автоматически создает таблицы и [схему базы данных](/guides/models/entities#automatic-schema) в зависимости от данных сущностей.

Иногда приходится работать с уже созданной базой данных. Тогда необходимо произвести несколько других операций.

Допустим, у нас есть база данных с таблицей:

```sql
CREATE TABLE t_operator (
    operator_id integer NOT NULL,
    s_name text
);
```

Тогда мы должны подготовить репозиторий следующим образом:

```ruby
# lib/bookshelf/repositories/operator_repository.rb
class OperatorRepository < Hanami::Repository
  self.relation = :t_operator

  mapping do
    attribute :id,   from: :operator_id
    attribute :name, from: :s_name
  end
end
```

При этом сущность не потребует никаких модификаций:

```ruby
# lib/bookshelf/entities/operator.rb
class Operator < Hanami::Entity
end
```

---

А сущность сможет использовать получившийся репозиторий:

```ruby
operator = Operator.new(name: "Jane")
operator.name # => "Jane"
```

Атрибуты будут вести себя так же, как и в общем случае:

```ruby
operator = OperatorRepository.new.create(name: "Jane")
  # => #<Operator:0x007f8e43cbcea0 @attributes={:id=>1, :name=>"Jane"}>
```

## Количество записей

Подсчет количества записей в таблице поддерживается большинством реляционных баз данных.

Для этой функции можно определить подобный метод в репозитории:

```ruby
class BookRepository < Hanami::Repository
  def count
    books.count
  end
end
```

Или реализовать его с дополнительными условиями:

```ruby
class BookRepository < Hanami::Repository
  # ...

  def on_sale_count
    books.where(on_sale: true).count
  end
end
```

Можно также воспользоваться самим SQL:

```ruby
class BookRepository < Hanami::Repository
  # ...

  def old_books_count
    books.read("SELECT id FROM books WHERE created_at < (NOW() - 1 * interval '1 year')").count
  end
end
```
