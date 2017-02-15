---
title: Руководство - Ассоциации
---

# Ассоциации

Ассоциации — это логическая связь между двумя сущностями.

<p class="warning">
  Текущая версия Hanami поддерживает ассоциации в качестве экспериментальной возможности и только для SQL адаптера.
</p>

## Устройство

Ассоциации состоят из соединенных записей в базе данных, поэтому мы сделали их частью репозиториев.

### Явный интерфейс

Когда мы задаем ассоциацию, репозиторий **не получит** дополнительных методов для своего открытого интерфейса.
Такое решение принято ради предотвращения засорения интерфейса методами, которые почти не будут использоваться.

<p class="notice">
  Репозитории по умолчанию не получают открытых методов для ассоциаций.
</p>

Если в нашем приложении необходима возможность создавать записи с ассоциациями, то метод для этого придется определить явно.

### Явный доступ

Тот же принцип работает и для методов доступа. Если мы хотим получать ассоциированные записи, то нужно явно задать метод реализующий эту возможность.

Если мы этого не сделаем, то вызов методов доступа всегда будет возвращать `nil`.

### Отказ от методов-посредников

Мы рекомендуем производить операции с ассоциациями только через методы определенные в репозиториях явно.
Hanami намеренно **не поддерживает** подобные методы:

  * `author.books` (чтобы получить все книги автора из базы данных);
  * `author.books.where(on_sale: true)` (чтобы получить все книги, имеющиеся в продаже);
  * `author.books << book` (чтобы ассоциировать книгу с автором);
  * `author.books.clear` (чтобы убрать ассоциации книг с автором).

Не забывайте, что `author.books` является обыкновенным массивом и изменения в нем **не отразятся на базе данных**.

## Типы ассоциаций

### Ассоциация Has Many

Также известна как _один-ко-многим_. Является ассоциацией одной сущности (`Author`) с коллекцией других сущностей(`Book`).

```shell
% bundle exec hanami generate migration create_authors
      create  db/migrations/20161115083440_create_authors.rb
```

```ruby
# db/migrations/20161115083440_create_authors.rb
Hanami::Model.migration do
  change do
    create_table :authors do
      primary_key :id

      column :name,       String,   null: false
      column :created_at, DateTime, null: false
      column :updated_at, DateTime, null: false
    end
  end
end
```

```shell
% bundle exec hanami generate migration create_books
      create  db/migrations/20161115083644_create_books.rb
```

```ruby
# db/migrations/20161115083644_create_books.rb
Hanami::Model.migration do
  change do
    create_table :books do
      primary_key :id
      foreign_key :author_id, :authors, on_delete: :cascade, null: false

      column :title,      String,   null: false
      column :created_at, DateTime, null: false
      column :updated_at, DateTime, null: false
    end
  end
end
```

```shell
% bundle exec hanami db prepare
```

```shell
% bundle exec hanami generate model author
      create  lib/bookshelf/entities/author.rb
      create  lib/bookshelf/repositories/author_repository.rb
      create  spec/bookshelf/entities/author_spec.rb
      create  spec/bookshelf/repositories/author_repository_spec.rb

% bundle exec hanami generate model book
      create  lib/bookshelf/entities/book.rb
      create  lib/bookshelf/repositories/book_repository.rb
      create  spec/bookshelf/entities/book_spec.rb
      create  spec/bookshelf/repositories/book_repository_spec.rb
```

Давайте добавим к `AuthorRepository` следующий код:

```ruby
# lib/bookshelf/repositories/author_repository.rb
class AuthorRepository < Hanami::Repository
  associations do
    has_many :books
  end

  def create_with_books(data)
    assoc(:books).create(data)
  end

  def find_with_books(id)
    aggregate(:books).where(id: id).as(Author).one
  end
end
```

Мы [явно объявили методы](#Явный интерфейс) только для тех операций, которые точно будут использоваться в модели.
Таким образом, мы избавили `AuthorRepository` от множества методов, которые никогда не будут использованы.

Давайте создадим автора вместе с коллекцией книг **за один запрос к базе данных**:

```ruby
repository = AuthorRepository.new

author = repository.create_with_books(name: "Alexandre Dumas", books: [{title: "The Count of Montecristo"}])
  # => #<Author:0x007f811c415420 @attributes={:id=>1, :name=>"Alexandre Dumas", :created_at=>2016-11-15 09:19:38 UTC, :updated_at=>2016-11-15 09:19:38 UTC, :books=>[#<Book:0x007f811c40fe08 @attributes={:id=>1, :author_id=>1, :title=>"The Count of Montecristo", :created_at=>2016-11-15 09:19:38 UTC, :updated_at=>2016-11-15 09:19:38 UTC}>]}>

author.id
  # => 1
author.name
  # => "Alexandre Dumas"
author.books
  # => [#<Book:0x007f811c40fe08 @attributes={:id=>1, :author_id=>1, :title=>"The Count of Montecristo", :created_at=>2016-11-15 09:19:38 UTC, :updated_at=>2016-11-15 09:19:38 UTC}>]
```

Что произойдет если мы попытаемся найти автора при помощи `AuthorRepository#find`?

```ruby
author = repository.find(author.id)
  # => #<Author:0x007f811b6237e0 @attributes={:id=>1, :name=>"Alexandre Dumas", :created_at=>2016-11-15 09:19:38 UTC, :updated_at=>2016-11-15 09:19:38 UTC}>
author.books
  # => nil
```

Мы не определили этого [метода доступа явно](#Явный доступ), поэтому `author.books` вернет `nil`.
Но мы можем использовать определенный ранее метод `#find_with_books`:

```ruby
author = repository.find_with_books(author.id)
  # => #<Author:0x007f811bbeb6f0 @attributes={:id=>1, :name=>"Alexandre Dumas", :created_at=>2016-11-15 09:19:38 UTC, :updated_at=>2016-11-15 09:19:38 UTC, :books=>[#<Book:0x007f811bbea430 @attributes={:id=>1, :author_id=>1, :title=>"The Count of Montecristo", :created_at=>2016-11-15 09:19:38 UTC, :updated_at=>2016-11-15 09:19:38 UTC}>]}>

author.books
  # => [#<Book:0x007f811bbea430 @attributes={:id=>1, :author_id=>1, :title=>"The Count of Montecristo", :created_at=>2016-11-15 09:19:38 UTC, :updated_at=>2016-11-15 09:19:38 UTC}>]
```

И на этот раз `author.books` вернет коллекцию ассоциированных записей.

---

Что делать если необходимо добавить или удалить книгу конкретного автора?
Определить новый метод:

```ruby
# lib/bookshelf/repositories/author_repository.rb
class AuthorRepository < Hanami::Repository
  # ...

  def add_book(author, data)
    assoc(:books, author).add(data)
  end
end
```

Воспользуемся этим методом:

```ruby
book = repository.add_book(author, title: "The Three Musketeers")
```

Аналогично для удаления:

```ruby
BookRepository.new.delete(book.id)
```
