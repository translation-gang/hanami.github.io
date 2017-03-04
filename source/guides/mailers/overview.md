---
title: Руководство - Электронная почта
---

# Обзор

За обработку шаблонов для электронной почты и их доставку отвечают специальные объекты — мэйлеры(mailers).

Каждый такой объект должен быть связан **только с одним прецедентом использования(feature)**. 

Предположим, в приложении необходимо отправлять электронные письма для реализации нескольких функций: _"подтверждение адреса"_ и _"восстановление пароля"_.
Тогда потребуется создать два отдельных мэйлера `Mailers::ConfirmEmailAddress` и `Mailers::ForgotPassword`, **а не один общий** `UserMailer`.

## Простой мэйлер

В Ханами есть специальный генератор, который создаст мэйлер, два шаблона и файл тестов.

```shell
% hanami generate mailer welcome
    create  spec/bookshelf/mailers/welcome_spec.rb
    create  lib/bookshelf/mailers/welcome.rb
    create  lib/bookshelf/mailers/templates/welcome.html.erb
    create  lib/bookshelf/mailers/templates/welcome.txt.erb
```

Вот как будет выглядеть код мэйлера:

```ruby
# lib/bookshelf/mailers/welcome.rb
class Mailers::Welcome
  include Hanami::Mailer
end
```

<p class="convention">
  Все мэйлеры находятся в пространстве имен <code>Mailers</code>.
</p>

