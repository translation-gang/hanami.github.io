---
title: Guides - Action HTTP Caching
---

# HTTP кэширование

HTTP кэшированием мы называем набор методов для ускорения взаимодействия между клиентом и сервером, предназначенных для HTTP 1.1 и реализованных поставщиками браузеров.
Существует несколько заголовков запроса отвечающих за эти механизмы.

## Заголовок Cache Control

Действия предлагают специальный DSL для установки заголовка ответа `Cache-Control`.
Первым аргументом в метод `cache_control` передается директива, такая как `:public` или `"must-revalidate"`, а вторым набор опций, таких как `:max_age`.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action

    cache_control :public, max_age: 600
      # => Cache-Control: public, max-age: 600

    def call(params)
      # ...
    end
  end
end
```

## Заголовок Expires

Другим специальным заголовком является `Expires`.
Он может быть использован для поддержки совместимости со старыми браузерами, которые не понимаю `Cache-Control`.

Hanami предлагает решение проблемы со старыми браузерами отсылая оба заголовка.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action
    expires 60, :public, max_age: 300
      # => Expires: Mon, 18 May 2015 09:19:18 GMT
      #    Cache-Control: public, max-age: 300

    def call(params)
      # ...
    end
  end
end
```

## Условный GET-запрос

_Условный GET-запрос_ это механизм, позволяющий в два шага сообщить браузеру, изменился ли ресурс со времени последнего запрос.
Для первого запроса ответ будет включать специальный HTTP заголовок, который браузер будет использовать для следующего запроса.
При следующем запросе сервер вычислит значение, определяющие состояние ресурса, и если оно совпадет с тем, что пришло от клиента, то сервер отправит ответ со статусом `304` (Изменений нет).

### Идентификатор ETag

Первый способ сравнить версии ресурса это идентификатор(обычно это MD5 токен).
Давайте укажем его при помощи `fresh etag:`.

Если полученный идентификатор не совпадает с заголовком запроса `If-None-Match`, то сервер вернет ответ `200` вместе с `ETag`, в котором будет содержаться это значение.
Если же совпадает, то действие просто вернет ответ `304`.

```ruby
# apps/web/controllers/users/show.rb
module Web::Controllers::Users
  class Show
    include Web::Action

    def call(params)
      @user = UserRepository.new.find(params[:id])
      fresh etag: etag

      # ...
    end

    private

    def etag
      "#{ @user.id }-#{ @user.updated_at }"
    end
  end
end

# Случай 1. Значения не совпадают.
# GET /users/23
#  => 200, ETag: 84e037c89f8d55442366c4492baddeae

# Случай 2. Значения совпадают.
# GET /users/23, If-None-Match: 84e037c89f8d55442366c4492baddeae
#  => 304
```

### Последние изменения

Вторым способом является проверка временной метки `fresh last_modified:`.

Если временная метка не совпадает с заголовком запроса `If-Modified-Since`, то сервер вернет ответ `200` вместе с `ETag`, в котором будет содержаться актуальная временная метка.
Если же совпадает, то действие просто вернет ответ `304`.

```ruby
# apps/web/controllers/users/show.rb
module Web::Controllers::Users
  class Show
    include Web::Action

    def call(params)
      @user = UserRepository.new.find(params[:id])
      fresh last_modified: @user.updated_at

      # ...
    end
  end
end

# Случай 1. Временная метка не совпадает.
# GET /users/23
#  => 200, Last-Modified: Mon, 18 May 2015 10:04:30 GMT

# Случай 2. Временная метка совпадает.
# GET /users/23, If-Modified-Since: Mon, 18 May 2015 10:04:30 GMT
#  => 304
```
