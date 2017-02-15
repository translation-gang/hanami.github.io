---
title: Руководство - Пользовательские ORM
---

# Пользовательские ORM

Компоненты Hanami легко могут быть заменены.
Их высокая модульность позволяет использовать ORM по вашему выбору.

Вот как это можно сделать:

  1. В вашем `Gemfile` уберите строку с `hanami-model`, добавьте гем желаемой ORM и выполните `bundle install`.
  2. Удалите папку `lib/`(например, командой `rm -rf lib`).
  3. В файле `config/environment.rb` удалите строку `require_relative '../lib/bookshelf'` и блок кода `model` в  `Hanami.configure`.
  4. В файле `Rakefile` удалите строку `require 'hanami/rake_tasks'`.

Обратите внимание, после удаления `hanami-model` перестанут быть доступны [команды баз данных](/guides/command-line/database) и [миграции](/guides/migrations/overview).
