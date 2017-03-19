---
title: Руководство - Экшены: Параметры экшенов
---

# Параметры запроса

Параметры(в переменной params) берутся из Rack переменной env и передаются в качестве аргумента `#call.`
Они похожи на обычный хэш Руби, но с некоторыми особенностями.

## Источники

Источником параметров могут стать:

  * [Переменные пути](/guides/routing/basic-usage)(такие как `/books/:id`);
  * Строки запросов(такие как `/books?title=Hanami`);
  * Тело запроса(`POST` запрос к `/books`).

## Доступ

Чтобы получить доступ к значениям параметров мы можем использвать _оператор подписки_ `#[]`.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action

    def call(params)
      self.body = "Строка запроса: #{ params[:q] }"
    end
  end
end
```

Если теперь мы откроем в браузере `/dashboard?q=foo`, то увидим `Строка запроса: foo`.

### Символы

Параметры и вложенные параметры могут быть запрошены **только** при помощи символов.

```ruby
params[:q]
params[:book][:title]
```
Что произойдет теперь, если `:book` не будет отправлен в запросе?
Значение `params[:book]` будет равно `nil`, а значит мы не сможем получить доступ к `title`.
В таком случае интерпретатор Руби вернет исключение `NoMethodError`.

У нас есть безопасное решение этой проблемы: метод `#get`.
Он принимает список символов, которые представляют собой вложенную структуру согласно порядку перечисления.

```ruby
params.get(:book, :title)             # => "Hanami"
params.get(:unknown, :nested, :param) # => nil вместо NoMethodError
```

## Белый список

Чтобы продемонстрировать принцип работы белого списка, создадим новый экшн:

```shell
bundle exec hanami generate action web signup#create
```

Мы хотим дать пользователям возможность регистрации.
Для этого мы создали HTML форму, которая отправляет post-запрос в этот экшн. Затем он обрабатывает данные и сохраняет их в таблице `users`.
Эта таблица содержит столбец булевого типа `admin` для определения возможностей пользователя.

Злоумышленник сможет использовать это для получения прав администратора.

С нашей стороны будет легко предотвратить такой сценарий отбросив уязвимые параметры во время обработки запроса.
Всегда помните: **params является потенциальным источником уязвимости**.

Мы используем `.params` для составления структуры вложенных параметров.

```ruby
# apps/web/controllers/signup/create.rb
module Web::Controllers::Signup
  class Create
    include Web::Action

    params do
      required(:email).filled
      required(:password).filled

      required(:address).schema do
        required(:country).filled
      end
    end

    def call(params)
      puts params[:email]             # => "alice@example.org"
      puts params[:password]          # => "secret"
      puts params[:address][:country] # => "Italy"

      puts params[:admin]             # => nil
    end
  end
end
```

Тогда даже если `admin` и будет отправлен вместе с запросом, то он не будет доступен из `params`.

## Валидации

### Возможные прецеденты использования

В нашем примере (с названием _"Signup"_) мы хотим сделать `password` обязательным параметром.

Представим, что необходимо реализовать еще и дополнительную функцию: _"приглашения"_.
Существующий пользователь должен иметь возможность предложить кому-то присоединиться.
Приглашенные будут выбирать пароль после появления их записи в базе, а значит нужно будет создать запись `User` без пароля.

Таким образом, необходимо по-разному обработать эти два случая.
В долгосрочной перспективе поддержка такого решения может создать много лишних проблем.

```ruby
# Пример прямолинейного решения
class User
  attribute :password, presence: { if: :password_required? }

  private
  def password_required?
     !invited_user? && !admin_password_reset?
  end
end
```

Мы можем использовать валидации в виде набора правил, определяющих корректность данных для **каждого отдельного случая**.
Запись `User` может храниться и с паролем, и без него, **в зависимости от порядка действий** и маршрута, по которому запись к нам попала.

### Ограничения

Второй важной функцией валидаций является предотвращение попадания недействительных данных в нашу систему.
В архитектуре MVC слой модели находится **дальше всего** от пользовательского ввода.
Будет слишком расточительно проверять данные непосредственно перед записью данных в базу.

Если мы **проверим правильность данных на этапе формирования**, до начала потока операций, то нежелательные данные будут отсеяны так рано, как это возможно.

Мы не хотим продолжать, если данные некорректны.
Рассмотрим следующий метод.

```ruby
def expensive_computation(argument)
  return if argument.nil?
  # ...
end
```

### Использование

Мы можем фильтровать данные в зависимости от их типа, наличия, допустимого диапозона значений и других ограничений.

```ruby
# apps/web/controllers/signup/create.rb
module Web::Controllers::Signup
  class Create
    include Web::Action
    MEGABYTE = 1024 ** 2

    params do
      required(:name).filled(:str?)
      required(:email).filled(:str?, format?: /@/).confirmation
      required(:password).filled(:str?).confirmation
      required(:terms_of_service).filled(:bool?)
      required(:age).filled(:int?, included_in?: 18..99)
      optional(:avatar).filled(size?: 1..(MEGABYTE * 3)
    end

    def call(params)
      if params.valid?
        # ...
      else
        # ...
      end
    end
  end
end
```

Валидация параметров делегируется [Hanami::Validations](https://github.com/hanami/validations).
Для получения полного списка возможностей этого модуля следует обратиться к его документации.

## Конкретный класс

Предметно-ориентированный язык(DSL), используемый для описания параметров, интуитивно понятен и прост в освоении, но у него есть и недостатки, делающие его визуально неприятным и неудобным для модульных тестов.

В качестве альтернативы можно "развернуть" класс и передать его в качестве аргумента в `.param`.

```ruby
# apps/web/controllers/signup/my_params.rb
module Web::Controllers::Signup
  class MyParams < Web::Action::Params
    MEGABYTE = 1024 ** 2

    params do
      required(:name).filled(:str?)
      required(:email).filled(:str?, format?: /@/).confirmation
      required(:password).filled(:str?).confirmation
      required(:terms_of_service).filled(:bool?)
      required(:age).filled(:int?, included_in?: 18..99)
      optional(:avatar).filled(size?: 1..(MEGABYTE * 3)
    end
  end
end
```

```ruby
# apps/web/controllers/signup/create.rb
require_relative './my_params'

module Web::Controllers::Signup
  class Create
    include Web::Action
    params MyParams

    def call(params)
      if params.valid?
        # ...
      else
        # ...
      end
    end
  end
end
```

## Обработка тела запроса

Rack игнорирует тело запроса, кроме тех случаев, когда запрос приходит из формы.
Если мы имеем дело с данными в формате JSON, то тело запроса тоже не будет доступно в `params`.

```ruby
module Web::Controllers::Books
  class Create
    include Web::Action
    accept :json

    def call(params)
      puts params.to_h # => {}
    end
  end
end
```

```shell
curl http://localhost:2300/books      \
  -H "Content-Type: application/json" \
  -H "Accept: application/json"       \
  -d '{"book":{"title":"Hanami"}}'    \
  -X POST
```

Если же мы хотим сделать его доступным через `params`, то нам необходимо воспользоваться этим методом:

```ruby
# apps/web/application.rb
module Web
  class Application < Hanami::Application
    configure do
      # ...
      body_parsers :json
    end
  end
end
```

Теперь `params.get(:book, :title)` вернет `"Hanami"`.
