---
title: Guides - Request & Response
---

# Запрос

Чтобы получить доступ к метаданным HTTP запроса, у действий есть доступ к закрытому(private) объекту `request`, который получается из `Rack::Request`. Рассмотрим небольшой пример.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action

    def call(params)
      puts request.path_info      # => "/dashboard"
      puts request.request_method # => "GET"
      puts request.get?           # => true
      puts request.post?          # => false
      puts request.xhr?           # => false
      puts request.referer        # => "http://example.com/"
      puts request.user_agent     # => "Mozilla/5.0 Macintosh; ..."
      puts request.ip             # => "127.0.0.1"
    end
  end
end
```

<p class="warning">
  Создание экземпляра класса <code>request</code> для каждого HTTP запроса может привести к потерям производительности.
  В качестве альтернативы можно рассмотреть получение той же информации из закрытых методов, таких как <code>accepts?</code> или напрямую из  Rack <code>params.env</code>
</p>

# Ответ

В качестве неявно возвращаемого значения вызова `#call` выступает сериализованный `Rack::Response`(см. [#finish](http://rubydoc.info/github/rack/rack/master/Rack/Response#finish-instance_method)):

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action

    def call(params)
    end
  end
end

# Будет возвращено [200, {}, [""]]
```

В нем есть закрытые методы доступа, позволяющие в явной форме установить статус, заголовки и тело ответа.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action

    def call(params)
      self.status = 201
      self.body   = 'Ваш ресурс был создан'
      self.headers.merge!({ 'X-Custom' => 'OK' })
    end
  end
end

# Будет возвращено [201, { "X-Custom" => "OK" }, ["Ваш ресурс был создан"]]
```

В качестве сокращения можно использовать `#status`.

```ruby
# apps/web/controllers/dashboard/index.rb
module Web::Controllers::Dashboard
  class Index
    include Web::Action

    def call(params)
      status 201, "Ваш ресурс был создан"
    end
  end
end

# Будет возвращено [201, {}, ["Ваш ресурс был создан"]]
```
