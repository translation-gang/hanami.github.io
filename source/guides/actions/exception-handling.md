---
title: Guides - Action Exception Handling
---

# Обработка исключений

Действия поддерживают элегантный API для обработки исключений.
Его поведение изменяется в зависимости от текущего окружения Hanami и особенностей конфигурации.

## Поведение по умолчанию

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action

    def call(params)
      raise 'boom'
    end
  end
end
```

Исключения автоматически отлавливаются в режиме эксплуатации(production), но не в режиме разработки(development).
В режиме эксплуатации при возниковнении исключения приложение вернет ответ `500` (Внутренняя ошибка сервера). Во время разработки будет выведен стек вызовов и информация для отладки.

Такое поведение может быть изменено при помощи настройки `handle_exceptions` в файле `apps/web/application.rb`.

## Нестандартный HTTP статус

Когда нужно настроить для исключения нестандартный статус HTTP мы можем использовать специальный предметно-ориентированный язык(DSL).

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action
    handle_exception ArgumentError => 400

    def call(params)
      raise ArgumentError
    end
  end
end
```

`handle_exception` принимает хэш, в котором ключем становится исключение, которое необходимо обработать, а значением &mdash; код статуса HTTP ответа.
В нашем примере получив исключение `ArgumentError` будет возвращен ответ `400` (Ошибка запроса).

## Нестандартные обработчики

Если назначения статуса ответа недостаточно, то мы можем указать еще и обработчик исключения, чтобы получить полный контроль над ним.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class PermissionDenied < StandardError
    def initialize(role)
      super "Это действие доступно только администратору, ваш статус: #{ role }"
    end
  end

  class Index
    include Web::Action
    handle_exception PermissionDenied => :handle_permission_error

    def call(params)
      unless current_user.admin?
        raise PermissionDenied.new(current_user.role)
      end

      # ...
    end

    private
    def handle_permission_error(exception)
      status 403, exception.message
    end
  end
end
```

Если указать имя метода(в виде символа) как значение `handle_exception`, то этот метод будет использоваться для обработки исключения.
В примере выше мы хотели защитить действие от нежелательного доступа и разрешить его только администраторам.

Когда будет получено исключение `PermissionDenied` его обработку начнет метод `:handle_permission_error`.
Он **обязан** принимать аргумент `exception`&mdash;который станет экземпляром исключения внутри `#call`.

<p class="warning">
Определяя нестандартный обработчик исключения необходимо, чтобы он принимал аргумент <code>exception</code>.
</p>
