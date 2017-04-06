---
title: "Руководство - Командная строка: Ассеты"
---

# Командная строка

## Ассеты

Мы можем управлять ассетами через командную строку.

### Прекомпиляция

Эта команда удобна для **развертывания** приложения.

```shell
% bundle exec hanami assets precompile
```

Сначала она прекомпилирует и копирует все ассеты из приложения и внешних гемов в папку `public/assets/`.

Потом [сжимает](/guides/assets/compressors) все таблицы стилей и код JavaScript.

В конце она приписывает к имени каждого файла его контрольную сумму.
Это позвляет браузерам правильно кэшировать файлы.

Также она генерирует манифест с контрольными суммами всех ассетов.

```shell
% cat public/assets.json
{
  # ...
  "/assets/application.css":"/assets/application-9ab4d1f57027f0d40738ab8ab70aba86.css"
}
```

Он используется хэлперами ассетов для создания путей к этим файлам.

### Пример

Допустим, у нас есть проект с тремя приложениями: `admin`, `metrics` и `web`.

```ruby
# config/environment.rb
# ...
Hanami::Container.configure do
  mount Metrics::Application, at: '/metrics'
  mount Admin::Application,   at: '/admin'
  mount Web::Application,     at: '/'
end
```

Их ассеты хранятся в следующих папках:

  * Admin: `apps/admin/assets`;
  * Metrics: `apps/metrics/assets`;
  * Web: `apps/web/assets`, `apps/web/vendor/assets`.
  
 
Более того, все они имеют внешние зависимости в Ember.js, который поставляется с условным гемом `hanami-ember`.

```shell
% tree .
├── apps
│   ├── admin
│   │   ├── assets
│   │   │   └── js
│   │   │       ├── application.js
│   │   │       └── zepto.js
# ...
│   ├── metrics
│   │   ├── assets
│   │   │   └── javascripts
│   │   │       └── dashboard.js.es6
# ...
│   └── web
│       ├── assets
│       │   ├── images
│       │   │   └── bookshelf.jpg
│       │   └── javascripts
│       │       └── application.js
# ...
│       └── vendor
│           └── assets
│               └── javascripts
│                   └── jquery.js
# ...
```

Запуск `hanami assets precompile` приведет к следующему результату:

```shell
% tree public
public
├── assets
│   ├── admin
│   │   ├── application-28a6b886de2372ee3922fcaf3f78f2d8.js
│   │   ├── application.js
│   │   ├── ember-b2d6de1e99c79a0e52cf5c205aa2e07a.js
│   │   ├── ember-source-e74117fc6ba74418b2601ffff9eb1568.js
│   │   ├── ember-source.js
│   │   ├── ember.js
│   │   ├── zepto-ca736a378613d484138dec4e69be99b6.js
│   │   └── zepto.js
│   ├── application-d1829dc353b734e3adc24855693b70f9.js
│   ├── application.js
│   ├── bookshelf-237ecbedf745af5a477e380f0232039a.jpg
│   ├── bookshelf.jpg
│   ├── ember-b2d6de1e99c79a0e52cf5c205aa2e07a.js
│   ├── ember-source-e74117fc6ba74418b2601ffff9eb1568.js
│   ├── ember-source.js
│   ├── ember.js
│   ├── jquery-05277a4edea56b7f82a4c1442159e183.js
│   ├── jquery.js
│   └── metrics
│       ├── dashboard-7766a63ececc63a7a629bfb0666e9c62.js
│       ├── dashboard.js
│       ├── ember-b2d6de1e99c79a0e52cf5c205aa2e07a.js
│       ├── ember-source-e74117fc6ba74418b2601ffff9eb1568.js
│       ├── ember-source.js
│       └── ember.js
└── assets.json
```

<p class="convention">
  Результирующие папки в <code>public/assets</code> отражают структуру приложений проекта. Приложение по умолчанию называется <code>Web</code> и его ассеты хранятся прямо в <code>/</code> в папке <code>public/assets</code>, а их URL будут начинаться с <code>/assets</code> (Например: <code>/assets/application-28a6b886de2372ee3922fcaf3f78f2d8.js</code>).
  
  Аналогично, для приложения <code>Admin</code> из папки <code>/admin</code> ассеты будут храниться в <code>public/assets/admin</code> и доступны по URL с соответсвующим префиксом <code>/assets/admin/application-28a6b886de2372ee3922fcaf3f78f2d8.js</code>.
</p>
