---
title: Руководство - Тестирование маршрутов
---

# Тестирование

В Hanami встроены инструменты упрощающие написание модульных тестов для маршрутов.

## Генерация маршрутов

Мы можем протестировать сгенерированные маршруты. Создадим отдельный файл spec для этой цели.

В объекте `Web.routes` содержатся все маршруты для приложения с названием `Web`.

Он предоставляет методы для создания путей, которые принимают [имя маршрута](/guides/routing/basic-usage#named-routes) в виде символа.
Вот как могут выглядеть необходимые нам тесты.

```ruby
# spec/web/routes_spec.rb
RSpec.describe Web.routes do
  it 'generates "/"' do
    actual = described_class.path(:root)
    expect(actual).to eq '/'
  end

  it 'generates "/books/23"' do
    actual = described_class.path(:book, id: 23)
    expect(actual).to eq '/books/23'
  end
end
```

## Распознавание маршрутов

Мы можем подойти к этому с другой стороны: создав заглушку Rack env мы сможем протестировать, правильно ли распознан машрут.

```ruby
# spec/web/routes_spec.rb
RSpec.describe Web.routes do

  # ...

  it 'recognizes "GET /"' do
    env   = Rack::MockRequest.env_for('/')
    route = described_class.recognize(env)

    expect(route).to be_routable

    expect(route.path).to   eq '/'
    expect(route.verb).to   eq 'GET'
    expect(route.params).to eq({})
  end

  it 'recognizes "PATCH /books/23"' do
    env   = Rack::MockRequest.env_for('/books/23', method: 'PATCH')
    route = described_class.recognize(env)

    expect(route).to be_routable

    expect(route.path).to   eq '/books/23'
    expect(route.verb).to   eq 'PATCH'
    expect(route.params).to eq(id: '23')
  end

  it 'does not recognize unknown route' do
    env   = Rack::MockRequest.env_for('/foo')
    route = subject.recognize(env)

    expect(route).to_not be_routable
  end
end
```

Когда мы используем `.recognize`, маршрутизатор возвращает маршрут, который он распознал. Этот объект был создан исключительно для использования при написании тестов.
Он берет на себя заботу обо всей важной информации о запрашиваемом маршруте.
