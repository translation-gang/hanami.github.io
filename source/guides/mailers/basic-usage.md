---
title: Руководство - Мэйлеры: Использование
---

# Использование

В [предыдущем разделе](/guides/mailers/overview) мы сгенерировали мэйлер. Попробуем использовать его.

## Информация

Для начала, необходимо указать отправителя и получателя(или получателей) и заголовок письма.
Для этого внутри мэйлера доступны три обязательных метода: `.from`, `.to`, `.subject` и два необязательных: `.cc`, `.bcc`.

Каждый из них принимает в качестве аргумента строку, а `.to` может принимать массив строк, если получателей несколько.

```ruby
class Mailers::Welcome
  include Hanami::Mailer

  from    'noreply@bookshelf.org'
  to      'user@example.com'
  subject 'Welcome to Bookshelf'
end
```

<p class="warning">
  Методы <code>.from</code> и <code>.to</code> обязательны для отправки электронного письма.
</p>

<p class="notice">
  Заголовок письма можно не указывать, но это считается плохой практикой.
</p>

Иногда бывает удобно задать адрес отправителя непосредственно в коде, но чаще всего это будет не самым практичным способом.

Практичнее же будет передать **вместо строки символ**. Тогда будет вызван метод с именем, содержащимся в символе. Из него можно будет вернуть необходимые данные

```ruby
class Mailers::Welcome
  include Hanami::Mailer

  from    'noreply@bookshelf.org'
  to      :recipient
  subject :subject

  private

  def recipient
    user.email
  end

  def subject
    "Welcome #{ user.name }!"
  end
end
```

<p class="notice">
  Не существует общего соглашения, определяющего имена вспомогательных методов мэйлеров.
</p>

<p class="notice">
  Мы рекомендуем определять только закрытые вспомогательные методы, если они не будут использованы в шаблонах.
</p>

## Контекст

### Локальные переменные

В примере выше была использована переменная `user`. Откуда она взялась?

Аналогично [представлениям](/guides/views/basic-usage) мэйлеры могут принимать _аргументы_.

```ruby
u = User.new(name: 'Luca', email: 'luca@example.com')
Mailers::Welcome.deliver(user: u)
```

Их число не ограничено. Для передачи используется хэш, но внутри мэйлера будет создана локальная переменная.

Так переданный ключ `:user` будет доступен в локальной переменной `user` .

<p class="warning">
 Ключи <code>:format</code> и <code>:charset</code> используются в служебных целях.
</p>

### Область видимости

Все открытые методы мэйлеров будут доступны в шаблонах:

```ruby
# lib/bookshelf/mailers/welcome.rb
class Mailers::Welcome
  include Hanami::Mailer

  # ...

  def greeting
    "Ahoy"
  end
end
```

```erb
# lib/bookshelf/mailers/templates/welcome.html.erb
<h2><%= greeting %></h2>
```
