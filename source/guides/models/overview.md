---
title: Руководство - Обзор моделей
---

# Модели

Модели Hanami строятся на разделении бизнес-логики приложения([сущностей](/guides/models/entities)) и механизмов хранения данных([репозиториев](/guides/models/repositories)).
Этот принцип позволяет достигнуть упрощения интерфейсов объектов, что делает их быстрее и удобнее, а также способствует их повторному использованию.

## Использование

В нашем примере мы будем использовать базу данных PostgreSQL.

Сгенерируем модель:

```shell
% bundle exec hanami generate model book
      create  lib/bookshelf/entities/book.rb
      create  lib/bookshelf/repositories/book_repository.rb
      create  spec/bookshelf/entities/book_spec.rb
      create  spec/bookshelf/repositories/book_repository_spec.rb
```

Получим сущность:

```ruby
class Book < Hanami::Entity
end
```

и репозиторий:

```ruby
class BookRepository < Hanami::Repository
end
```

Потребуется также сгенерировать миграцию:

```shell
% bundle exec hanami generate migration create_books
      create  db/migrations/20161113154510_create_books.rb
```

Изменим файл миграции следующим образом:

```ruby
Hanami::Model.migration do
  change do
    create_table :books do
      primary_key :id
      column :title,      String
      column :created_at, DateTime
      column :updated_at, DateTime
    end
  end
end
```

Теперь подготовим базу данных:

```shell
% bundle exec hanami db prepare
```

На этом этапе мы уже можем использовать наш репозиторий:

```shell
% bundle exec hanami console
irb(main):001:0> book = BookRepository.new.create(title: "Hanami")
=> #<Book:0x007f95ccefb320 @attributes={:id=>1, :title=>"Hanami", :created_at=>2016-11-13 15:49:14 UTC, :updated_at=>2016-11-13 15:49:14 UTC}>
```

---

Далее вы можете подробнее узнать о [репозиториях](/guides/models/repositories), [сущностях](/guides/models/entities), [миграциях](/guides/migrations/overview), и [консольных командах баз данных](/guides/command-line/database).
