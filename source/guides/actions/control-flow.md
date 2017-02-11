---
title: Документация - Порядок выполнения
---

# Порядок выполнения

## Обратные вызовы

Если мы хотим реализовать некую логику до или после вызова `#call`, то можем использовать обратные вызовы.
Обратные вызовы полезны когда нужно разгрузить код от рутинных задач, таких как проверка прав пользователя.

За них отвечают DSL методы `before` и `after`.
Эти методы принимают символ с именем необходимого метода или анонимный объект proc.

### Методы в обратных вызовах

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action
    before :track_remote_ip

    def call(params)
      # ...
    end

    private
    def track_remote_ip
      @remote_ip = request.ip
      # ...
    end
  end
end
```

В коде выше мы отслеживаем IP адрес для стороннего сервиса аналитики.
Эта функция никак не связана с бизнес-логикой проекта, а значит ее легко можно поместить в обратный вызов.

Методы обратных вызовов могут принимать `params` в качестве аргумента.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action
    before :validate_params

    def call(params)
      # ...
    end

    private
    def validate_params(params)
      # ...
    end
  end
end
```

### Объекты proc в обратных вызовах

Пример выше может быть переписан с использованием анонимного объекта proc.
Он будет привязан непосредственно к контексту экшена.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action
    before { @remote_ip = request.ip }

    def call(params)
      # здесь можно использовать @remote_ip
      # ...
    end
  end
end
```

Объект proc может принимать необязательный аргумент `params`.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action
    before {|params| params.valid? }

    def call(params)
      # ...
    end
  end
end
```

<p class="warning">
Не стоит использовать обратные вызовы для функций, которые относятся к предметной области. Например, для отправки электронной почты.
Это считается антипаттерном и может привести к проблемам с дальнейшей поддержкой и тестированием кода, а так же другим непредсказуемым последствиям.
</p>

## Остановка

Использование механизма исключений является неоптимальным с точки зрения интерпретатора Ruby.
Для него есть более подходящая альтернатива, поддерживаемая языком: **сигналы** (см. `throw` и `catch`).

Hanami использует их для обеспечения более оптимального **управления порядком действий** в экшенах при помощи `#halt`.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action

    def call(params)
      halt 401 unless authenticated?
      # ...
    end

    private
    def authenticated?
      # ...
    end
  end
end
```

После использования этой инструкции, **поток действий прерывается** и управление передается фреймворку.
Код после нее будет пропущен.

<p class="warning">
Когда использована команда <code>halt</code>, поток действий прерывается и управление передается фреймворку.
</p>

Это значит, что `halt` можно использовать чтобы вообще пропустить `#call`, если эта команда будет выполнена в обратном вызове `before`.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action
    before :authenticate!

    def call(params)
      # ...
    end

    private
    def authenticate!
      halt 401 if current_user.nil?
    end
  end
end
```

`#halt` принимает код HTTP статуса в качестве аргумента.
При этом в теле ответа будет выведено соответствующее сообщение. В данном случае для 404: "Не авторизован".

В качестве необязательного второго аргумента можно передать значение тела ответа.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action

    def call(params)
      halt 404, "These aren't the droids you're looking for"
    end
  end
end
```

После использования `#halt` **Hanami** по умолчанию выведет страницу с соответствующим HTTP состоянием и его сообщением.

<p><img src="/images/default-template.png" alt="Hanami default template" class="img-responsive"></p>

Чтобы настроить отображение страницы для ошибки 404 вы можете использовать собственную [страницу ошибки](/guides/views/custom-error-pages).

## HTTP статус

В случае когда обработку исключения необходимо передать на уровень представления достаточно вместо `#halt` использовать `#status=`.

Типичным применением такой возможности является **обработка ошибки при заполнении формы**: мы хотим вернуть HTTP статус ошибки (`422`) и при этом позволить представлению отобразить форму еще раз с учетом допущенных ошибок.

```ruby
# apps/web/controllers/books/create.rb
module Web::Controllers::Books
  class Create
    include Web::Action

    params do
      required(:title).filled(:str?)
    end

    def call(params)
      if params.valid?
        # передача в хранилище
      else
        self.status = 422
      end
    end
  end
end
```

```ruby
# apps/web/views/books/create.rb
module Web::Views::Books
  class Create
    include Web::View
    template 'books/new'
  end
end
```

```erb
# apps/web/templates/books/new.html.erb
<% unless params.valid? %>
  <ul>
    <% params.error_messages.each do |error| %>
      <li><%= error %></li>
    <% end %>
  </ul>
<% end %>

<!-- здесь начинается форма -->
```

## Переадресация

Переадресация является особым способом обработки запроса.
Если мы хотим перенаправить запрос к другому ресурсу, то можем воспользоваться `redirect_to`.

После вызова `redirect_to` обработка запроса прекращается и **код после нее не исполняется**.

Этот метод принимает строку, представляющую URI, и необязательный аргумент `:status`.
По умолчанию статус становится `302`.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action

    def call(params)
      redirect_to routes.root_path
      foo('bar') # Эта строка никогда не исполнится
    end
  end
end
```

### На предыдущую страницу

Иногда необходимо вернуть пользователя на последнюю страницу из истории его браузера. Это можно сделать так:

```ruby
redirect_to request.headers["Referer"] || fallback_url
```
