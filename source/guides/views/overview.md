---
title: Документация - Обзор представлений
---

# Обзор

Представлением мы называем объект, который отвечает за отрисовку шаблона.

В общем случае в приложении Hanami входящие HTTP запросы направляются в [маршрутизатор](/guides/routing/overview). Он создает объект [экшена](/guides/actions/overview), ответственного за подготовку данных для ответа и выставление его заголовков и статуса.
В конце остается только создать тело ответа. Этим и займется слой представления.

## Простое представление

Hanami поставляется с генератором для экшенов, который также создает файл представления и шаблон.

```shell
% hanami generate action web dashboard#index
    insert  apps/web/config/routes.rb
    create  spec/web/controllers/dashboard/index_spec.rb
    create  apps/web/controllers/dashboard/index.rb
    create  apps/web/views/dashboard/index.rb
    create  apps/web/templates/dashboard/index.html.erb
    create  spec/web/views/dashboard/index_spec.rb
```

Обратим внимание на имена файлов. У нас будет экшн `Web::Controllers::Dashboard::Index` ([подробнее об именовании экшенов](/guides/actions/overview)).
Представление же получит похожее имя: `Web::Views::Dashboard::Index`.

Содержание файла будет таким:

```ruby
# apps/web/views/dashboard/index.rb
module Web::Views::Dashboard
  class Index
    include Web::View
  end
end
```

### Именование

Начало файла выглядит почти так же, как начало файла соответствующего [экшена](/guides/actions/overview).
Отличается только название модуля: `Views`, вместо `Controllers`.

**Все представления содержатся в этом модуле.**
Он создается во время запуска приложения.

<p class="convention">
  В приложении с именем <code>Web</code> все представления будут доступны через <code>Web::Views</code>.
</p>

**Это важное условие выполнения программы.**

Если явно не указано иное, то когда экшн выполнит свои инструкции, фреймворк попытается использовать соответствующий файл представления.

<p class="convention">
  Для экшена с именем <code>Web::Controllers::Home::Index</code> Hanami ожидает наличие представления с именем <code>Web::Views::Home::Index</code>.
</p>

### Модуль View

Все главные компоненты Hanami являются подключаемым модулями(mixins).
Hanami может включать несколько приложений в рамках одного процесса, а значит конфигурации компонентов этих приложений должны быть жестко разграничены.

В нашем примере есть строка кода `include Web::View`.
Она означает, что наше представление будет использовать конфигурацию приложения с именем `Web`.

<p class="convention">
  Для приложения с именем <code>Web</code> будет использоваться модуль <code>Web::View</code>.
</p>
