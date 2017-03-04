---
title: Руководство - Разделение кода мэйлерами
---

# Разделение кода

## Метод prepare

В настройках приложения(`apps/web/application.rb`) есть участок кода, разделяемый **всеми мэйлерами** в нашем приложении. 
Когда любой мэйлер подключит `Web::Mailer`, будет вызван этот участок кода в контексте данного мэйлера.

Этот метод основан на механизме работы Ruby модулей и называется обратным вызовом `included`.

Представим, что мы хотим установить для всех мэйлеров отправителя по умолчанию.
Вместо явного указания внутри каждого из мэйлеров мы можем последовать принципу DRY.

Создадим модуль:

```ruby
# lib/mailers/default_sender.rb
module Mailers::DefaultSender
  def self.included(mailer)
    mailer.class_eval do
      from 'sender@bookshelf.org'
    end
  end
end
```

А затем включим его во все мэйлеры посредством `prepare`:

```ruby
# lib/bookshelf.rb
# ...
Hanami::Mailer.configure do
  # ...
  prepare do
    include Mailers::DefaultSender
  end
end.load!
```

<p class="warning">
Код включенный посредством <code>prepare</code> будет доступен во всех мэйлерах приложения.
</p>

