---
title: "Руководство - Командная строка: Генераторы"
---

# Командная строка

## Генераторы

Ханами предлагает удобные генераторы кода для ускорения процесса разработки.

### Генератор приложения

Архитектура Ханами позволяет создать несколько приложений Ханами в одном проекте в папке `apps/`.
Приложение по умолчанию называется `Web` и расположено в `apps/web`.

Мы можем сгенерировать новые приложения следующим образом:

```shell
% bundle exec hanami generate app admin
```

Будет сгенерировано приложение `Admin` в папке `apps/admin`.

### Генератор экшенов

Генератов экшенов помимо экшена создаст соответствующее представление, шаблон, путь и тест. Все за одну команду.

```shell
% bundle exec hanami generate action web books#show
```

Первый аргумент, `web`, это имя желаемо приложения внутри проекта.

Аргумент `books#show` состоит из имени контроллера и экшена, разделенных знаком(`#`).

Если необходимо сгенерировать только экшн, без представления и шаблона, то следует использовать аргумент `--skip-view`.

```shell
% bundle exec hanami generate action web books#show --skip-view
```

Если необходимо сгенерировать экшн и указать HTTP глагол, то следует использовать аргумент `--method`.

```shell
% bundle exec hanami generate action web books#create --method=post
```

#### Маршрут

После генерации контроллера создается маршрут.

```ruby
# apps/web/config/routes.rb
get '/books', to: 'books#show'
```

Если мы хотим настроить URL маршрута из командной строки, то можем передать аргумент `--url`.

```shell
% bundle exec hanami generate action web books#show --url=/books/:id
```

Будет сгенерирован следующий маршрут:

```ruby
# apps/web/config/routes.rb
get '/books/:id', to: 'books#show'
```

По умолчанию маршруты создаются для HTTP метода `GET`, если имя экшена не:

- `create`, для которого будет создан `POST`
- `update`, для которого будет создан `PATCH`
- `destroy`, для которого будет создан `DELETE`

Это поможет вам быстрее создавать [ресурсы в стиле REST](/guides/routing/restful-resources).

Также вы все еще сможете явно указать HTTP метод передав во время вызова `hanami generate action` аргумент `--method`.

### Генератор моделей

При помощи одной команды можно сгенерировать сущность и связанный репозиторий:

```shell
% bundle exec hanami generate model book
      create  lib/bookshelf/entities/book.rb
      create  lib/bookshelf/repositories/book_repository.rb
      create  spec/bookshelf/entities/book_spec.rb
      create  spec/bookshelf/repositories/book_repository_spec.rb
```

Также будут сгенерированы файлы тестов.

### Генератор миграций

Позволяет сгенерировать файл миграции базы данных.

```shell
% bundle exec hanami generate migration create_books
      create  db/migrations/20161112113203_create_books.rb
```

Будет сгенерирована пустая миграция с временной отметкой и указанным именем: `db/migrations/20161112113203_create_books.rb`.

### Генератор мэйлеров

Позволяет сгенерироват мэйлер.

```shell
% bundle exec hanami generate mailer welcome
```

Создаст следующие файлы:

```shell
% tree lib/
lib
├── bookshelf
│   # ...
│   ├── mailers
│   │   ├── templates
│   │   │   ├── welcome.html.erb
│   │   │   └── welcome.txt.erb
│   │   └── welcome.rb # Mailers::Welcome
# ...
```

### Генератор секретного токена

Позволяет сгенерировать секретный токен для поддержки HTTP сессий отдельного приложения.

```shell
% bundle exec hanami generate secret web
Set the following environment variable to provide the secret token:
WEB_SESSIONS_SECRET="a6aa65a71538a56faffe1b1c9e96c0dc600de5dd14172f03c35cc48c3b27affe"
```
