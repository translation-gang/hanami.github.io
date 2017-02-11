---
title: Документация - Доступ к переменным экшенов
---

# Представления

В ряде случаев мы хотим показать пользователю данные полученные на уровне модели.
Hanami решает эту проблему просто: данные не попадут из контроллера в представление, пока мы прямо это не укажем.

Мы используем простой и мощный механизм для достижения этой цели: _**выставления(exposures)**_.

Выставления получают имена переменных из контроллера и создают для них _методы доступа_ в представлениях.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action
    expose :greeting

    def call(params)
      @greeting = "Hello"
      @foo      = 23
    end
  end
end
```
В этот примере мы передали `:greeting`, но не `:foo`.
Представление и шаблоны смогут использовать только `greeting`.

```ruby
# apps/web/views/dashboard/index.rb
module Web::Views::Dashboard
  class Index
    include Web::View

    def welcome_message
      greeting + " and welcome"
    end
  end
end
```

Если же мы попытаемся вызвать `foo`, то Ruby выдаст ошибку `NoMethodError`.
