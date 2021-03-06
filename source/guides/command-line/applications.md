---
title: "Руководство - Командная строка: Приложения"
---

# Командная строка

## Приложения

Сгенерировать новый проект можно командой `hanami new`, за которой следует желаемое имя проекта.

```shell
% hanami new bookshelf
```

### Архитектура

По умолчанию проект генерируется с архитектурой _"Контейнер"_.

Мы можем передать аргумент `--architecture`(сокращенно `--arch`) чтобы указать другую архитектуру.

На текущий момент поддерживаются два типа архитектуры:

  * `container` (по умолчанию)
  * `app`

Следующая команда сгенерирует проект `admin`, использующий архитектуру _"Приложение"_.

```shell
% hanami new admin --arch=app
```

### База данных

По умолчанию в качестве хранилища данных используется "игрушечная" база данных на файловой системе.
Это сделано для того, чтобы обеспечить разработчиков быстрым инструментом для прототипирования.

Мы можем передать аргумент `--database`, чтобы Ханами сгенерировал файлы для соответствующей базы данных.

Поддерживаются:

  * `filesystem` (по умолчанию)
  * `memory`
  * `postgres`
  * `postgresql`
  * `sqlite`
  * `sqlite3`
  * `mysql`

### Фреймворк тестирования

По умолчанию в качестве фреймворка для тестирования используется Minitest.

Мы можем передать аргумент `--test`, чтобы указать другой фреймворк из списка ниже:

  * `minitest` (default)
  * `rspec`
