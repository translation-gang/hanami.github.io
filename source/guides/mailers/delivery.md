---
title: Руководство - Доставка почты
---

# Доставка

## Multipart Delivery

По умолчанию мэйлер составляет письма из двух частей. Одна включает в себя разметку HTML, а вторая текст.
Именно поэтому генератор создает два файла шаблона.

Чтобы обработать обе эти части и доставить их, необходимо использовать метод:

```ruby
Mailers::Welcome.deliver
```

Мэйлеры Ханами гибко подстраиваются под ряд сценариев.

## Обработка единственной составляющей

Предположим, нам необходимо отправить письмо только из HTML или текстового файла.
Тогда мы можем указать это при вызове метода доставки:

```ruby
Mailers::Welcome.deliver(format: :html)
# или
Mailers::Welcome.deliver(format: :txt)
```

В этом случае будет обработан только один шаблон.

## Удаление шаблонов

Если в приложении используется только одна составляющая шаблона, то из него можно без последствий удалить вторую.

<p class="warning">
  Для отправки письма достаточно одного шаблона.
</p>

## Конфигурация

Для определения шлюза отправки электронных писем используется конфигурация`delivery`.

### Встроенные методы

Способ доставки определяется соответствующим символом:

  * Exim (`:exim`);
  * Sendmail (`:sendmail`);
  * SMTP (`:smtp`, для локального SMTP);
  * SMTP соединение (`:smtp_connection`, посредством `Net::SMTP` - для удаленного SMTP);
  * Test (`:test`, для тестов).

По умолчанию используется SMTP (`:smtp`) в режиме эксплуатации(production) и `:test` в режиме разработки и тестирования.

Вторым необязательным аргументов является остальная конфигурация:

```ruby
# lib/bookshelf.rb
# ...
Hanami::Mailer.configure do
  # ...
  delivery do
    development :test
    test        :test
    production  :smtp,
      address:              "smtp.gmail.com",
      port:                 587,
      domain:               "bookshelf.org",
      user_name:            ENV['SMTP_USERNAME'],
      password:             ENV['SMTP_PASSWORD'],
      authentication:       "plain",
      enable_starttls_auto: true
  end
end.load!
```

Для более сложных конфигураций мы рекомендуем обратиться к документации [гема "mail"](https://github.com/mikel/mail).
В низкоуровневой основе **Hanami::Mailer** лежит именно эта библиотека.

А так как Ханами использует гем `mail`, который _де-факто_ является стандартом для Руби приложений, то мы имеем полную совместимость с шлюзами наиболее популярных поставщиков.
[Sendgrid](https://devcenter.heroku.com/articles/sendgrid#ruby-rails), [Mandrill](https://devcenter.heroku.com/articles/mandrill#sending-with-smtp), [Postmark](https://devcenter.heroku.com/articles/postmark#sending-emails-via-the-postmark-smtp-interface) и [Mailgun](https://devcenter.heroku.com/articles/mailgun#sending-emails-via-smtp) это одни из нескольких, использующих SMTP и имеющих подробные руководства.

### Нестандартные методы 

Если необходимо наладить нестандартный способ доставки почты, то можно передать соответствующий класс в конфигурацию. 

Например, так можно использовать [Mandrill API](https://mandrillapp.com/api/docs/) для доставки электронной почты.

```ruby
# lib/bookshelf.rb
# ...
require 'lib/mailers/mandrill_delivery_method'

Hanami::Mailer.configure do
  # ...
  delivery do
    production MandrillDeliveryMethod, api_key: ENV['MANDRILL_API_KEY']
  end
end.load!
```

Объект должен отвечать на сообщения `#initialize(options = {})` и`#deliver!(mail)`, где `mail` экземпляр класса [`Mail::Message`](https://github.com/mikel/mail/blob/master/lib/mail/mail.rb).

```ruby
class MandrillDeliveryMethod
  def initialize(options)
    @api_key = options.fetch(:api_key)
  end

  def deliver!(mail)
    send convert(mail)
  end

  private

  def send(message)
    gateway.messages.send message
  end

  def convert(mail)
    # Convert a Mail::Message instance into a Hash structure
    # See https://mandrillapp.com/api/docs/messages.ruby.html
  end

  def gateway
    Mandrill::API.new(@api_key)
  end
end
```

<p class="notice">
  Обратите внимание, что этот пример иллюстрирует только один из очень специфических вариантов. Если вы хотите использовать Mandrill, более предпочтительным выбором будет SMTP.
</p>
