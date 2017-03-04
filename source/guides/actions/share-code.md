---
title: Руководство - Общий код
---

# Общий код

Экшены как объекты имеют множество преимуществ, но они делают разделение кода менее интуитивным.
Здесь мы рассмотрим несколько методов, которые решают эту проблему.

## Метод prepare

В настройках приложения(`apps/web/application.rb`) есть участок кода, разделяемый **всеми экшенами** в нашем приложении. Когда любой экшн подключит `Web::Action`, будет вызван этот участок кода в контексте данного экшена.

Этот метод основан на механизме работы Ruby модулей и называется обратным вызовом `included`.

Представим, что нам необходимо проверить авторизацию пользователя, от которого пришел запрос.

Мы создаем модуль `apps/web/controllers/authentication.rb`.

```ruby
# apps/web/controllers/authentication.rb
module Web
  module Authentication
    def self.included(action)
      action.class_eval do
        before :authenticate!
        expose :current_user
      end
    end

    private

    def authenticate!
      halt 401 unless authenticated?
    end

    def authenticated?
      !!current_user
    end

    def current_user
      @current_user ||= UserRepository.new.find(session[:user_id])
    end
  end
end
```

Он должен для каждого запроса выполнять [обратный вызов before](/guides/actions/control-flow), внутри которого вызывается `:authenticate!`.
Если пользователь не будет авторизрован, то будет возвращен ответ `401`, а иначе вызов пойдет дальше и достигнет `#call`.
Вдобавок `current_user` становится доступен во всех представлениях(см. [Представления](/guides/actions/exposures)).

Это было бы действительно утомительно включать этот модуль в каждый экшн в приложении.

Мы можем использовать `controller.prepare` чтобы включить его для всех экшенов сразу.

```ruby
# apps/web/application.rb
require_relative './controllers/authentication'

module Web
  class Application < Hanami::Application
    configure do
      controller.prepare do
        include Web::Authentication
      end
    end
  end
end
```

<p class="warning">
Код включенный в <code>prepare</code> доступен во всех экшенах приложения.
</p>

### Пропуск обратного вызова

Допустим мы уже подключили `Authentication` глобально, но хотим пропустить его вызов для некоторых ресурсов.
Типичным примером подобной ситуации может служить переадресация неавторизованного пользователя к форме входа.

Решение этой проблемы очень простое и элегантное: переопределить метод.

```ruby
# apps/web/controllers/sessions/new.rb
module Web::Controllers::Sessions
  class New
    include Web::Action

    def call(params)
      # ...
    end

    private
    def authenticate!
      # ничего не выполняется
    end
  end
end
```

Экшн все еще будет вызывать `:authenticate!`. Технически, **выполнение обратного вызова невозможно пропустить**.
Но если мы переопределим этот метод на пустой, то во время его вызова ничего не произойдет и наш неавторизованный пользователь сможет получить запрошенный ресурс.

## Включение модулей

Представим, что у нас есть ресурс в стиле REST с именем `books`.
Есть так же несколько действий(`show`, `edit`, `update` и `destroy`), которым необходимо находить отдельные книги для выполнения своих функций.
Что если мы захотим уменьшить количество повторений в таком коде?
Ruby поможет нам сделать это.

```ruby
# apps/web/controllers/books/set_book.rb
module Web::Controllers::Books
  module SetBook
    def self.included(action)
      action.class_eval do
        before :set_book
      end
    end

    private

    def set_book
      @book = BookRepository.new.find(params[:id])
      halt 404 if @book.nil?
    end
  end
end
```

Мы определили модуль с общим поведением. Осталось только включить его в необходимые экшены.

```ruby
# apps/web/controllers/books/update.rb
require_relative './set_book'

module Web::Controllers::Books
  class Update
    include Web::Action
    include SetBook

    def call(params)
      # ...
    end
  end
end
```
