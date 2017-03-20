---
title: Руководство - Мэйлеры: Тестирование
---

# Тестирование

Мы не хотим, чтобы электронные письма из среды разработки и тестирования попали во внешний мир.
Для этих двух видов окружения [метод доставки](/guides/mailers/delivery) должен быть установлен как `:test`.

Чтобы убедиться, что электронные письма отправляются согласно ожиданиям, мы можем посмотреть в `Hanami::Mailer.deliveries`.
Это массив писем, которые приложение отправило бы в режиме эксплуатации.
Перед тестированием следует убедиться, что этот массив очищен.

```ruby
# spec/bookshelf/mailers/welcome_spec.rb
require 'spec_helper'

describe Mailers::Welcome do
  before do
    Hanami::Mailer.deliveries.clear
  end

  let(:user) { ... }

  it "delivers welcome email" do
    Mailers::Welcome.deliver(user: user)
    mail = Hanami::Mailer.deliveries.last

    mail.to.must_equal             [user.email]
    mail.body.encoded.must_include "Hello, #{ user.name }"
  end
end
```
