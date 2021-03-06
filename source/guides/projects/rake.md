---
title: Руководство - Проекты: Rake-таски
---

# Rake-таски

Ханами поставляется с набором стандартных Rake-тасков. Разработчики могут использовать их в качестве _предпосылок(prerequisites)_ для написания собственных задач.

```shell
% bundle exec rake -T
rake environment # Загрузка всего проекта
rake test        # Загрузка тестов (гем Minitest)
rake spec        # Загрузка тестов (гем RSpec)
```

## Окружение

Используйте эти Rake-таски когда потребуется получить доступ к коду проекта(сущностям, действиям, представлениям и др.).

### Пример

Необходимо создать Rake-таски для доступа к коду репозиториев проекта.

```ruby
# Rakefile

task clear_users: :environment do
  UserRepository.new.clear
end
```

```shell
bundle exec rake clear_users
```

## Тесты и спецификации

Это стандартные Rake-таски, которые запускают набор тестов.

Запуск следующих команд приводит к одинаковому результату.

```shell
% bundle exec rake
```

```shell
% bundle exec rake test
```

<p class="convention">
  Rake-таски <code>:test</code> (и <code>:spec</code>) эквивалентны.
</p>

## Совместимость с хостингами

Многие сервисы хостинга для Руби серверов, организованные в соответствии с моделью SaaS были спроектированы так, чтобы удовлетворять потребности приложений Ruby On Rails. В частности, Heroku ожидает в любом Руби приложении найти следующие стандартные для Ruby On Rails таски Rake:

  * `db:migrate`
  * `assets:precompile`

Heroku не позволяет развернуть приложение с другой конфигурацией. Поэтому мы поддерживаем такие "стандартные" Rake-таски из Ruby On Rails.

**Если обстоятельства позволяют, не полагайтесь на них, а используйте [командную строку](/guides/command-line/database) `hanami` вместо них.**
