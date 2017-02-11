---
title: Документация - Тестирование экшенов
---

# Тестирование

Hanami уделяет особое внимание возможности тестирования кода и предлагает для него продвинутые возможности.
Фреймворк поддерживает Minitest(по умолчанию) и Rspec.

## Модульные тесты

Прежде всего, экшены могут быть подвергнуты модульному тестированию.
Это значит, что их можно отделить от остального кода, поместить в разные условия и проверить соотвествует ли их поведение ожидаемому. Все это непосредственно с **экземплярами объектов экшенов**.

```ruby
# spec/web/controllers/dashboard/index_spec.rb
require 'spec_helper'
require_relative '../../../../apps/web/controllers/dashboard/index'

describe Web::Controllers::Dashboard::Index do
  let(:action) { Web::Controllers::Dashboard::Index.new }
  let(:params) { Hash[] }

  it "is successful" do
    response = action.call(params)
    response[0].must_equal 200
  end
end
```

В примере выше `action` это экземпляр `Web::Controllers::Dashboard::Index`, для которого мы можем вызвать `#call` с передачей ему параметров.

[Неявно возвращаемое значение](/guides/actions/rack-integration) является сериализованным Rack ответом.
Мы проверяем, что код состояния ответа(`response[0]`) будет равен `200`.

### Запуск тестов

Мы можем запустить как весь набор тестов, так и отдельный файл.

В первом случае нам необходима стандартная Rake задача `bundle exec rake`.
Все зависимости и код приложения(экшены, представления, сущности и др.) будут полностью загружены.
**В этом случае время загрузки будет больше.**

<p class="notice">
Весь комплект тестов может быть запущен через стандартную задачу Rake. Тогда будут загружены все зависимости и код всего приложения.
</p>

Во втором случае мы можем запустить: `ruby -Ispec spec/web/controllers/dashboard/index_spec.rb` (или `rspec spec/web/controllers/dashboard/index_spec.rb` для RSpec).
Когда мы запускаем отдельный файл **загружаются только необходимые части приложения**.

Обратите внимание на `require_relative` в примере.
Эта строка **сгенерирована автоматически** и она необходима для загрузки отдельного экшена в тест.
Этот механизм позволяет запускать тесты **изолированно**.
**В этом случае время загрузки будет минимальным**.

<p class="notice">
Модульные тесты могут быть запущены по отдельности. Они загрузят только свои зависимости без всего приложения.
Тестируемый класс будет загружен через строку <code>require_relative</code>, сгенерированную автоматически.
В этом случае тесты будут проводиться значительно быстрее.
</p>

### Параметры

Во время тестирования экшенов мы легко можем имитировать параметры и заголовки запросов.
Необходимо только передать хэш.
Заголовки для Rack env, такие как `HTTP_ACCEPT`, могут быть переданы вместе с такими параметрами как `:id`.

Следующий пример использует оба варианта.

```ruby
# spec/web/controllers/users/show_spec.rb
require 'spec_helper'
require_relative '../../../../apps/web/controllers/users/show'

describe Web::Controllers::Users::Show do
  let(:action)  { Web::Controllers::Users::Show.new }
  let(:format)  { 'application/json' }
  let(:user_id) { '23' }

  it "is successful" do
    response = action.call(id: user_id, 'HTTP_ACCEPT' => format)

    response[0].must_equal                 200
    response[1]['Content-Type'].must_equal "#{ format }; charset=utf-8"
    response[2].must_equal                 ["ID: #{ user_id }"]
  end
end
```

Тестируемый код.

```ruby
# apps/web/controllers/users/show.rb
module Web::Controllers::Users
  class Show
    include Web::Action

    def call(params)
      puts params.class # => Web::Controllers::Users::Show::Params
      self.body = "ID: #{ params[:id] }"
    end
  end
end
```

<p class="notice">
Имитирование параметров и заголовков запроса в случае Hanami происходит просто. Мы передаем их как <code>Hash</code> и они превращаются в <code>Hanami::Action::Params</code>.
</p>

### Доступ к внутренним переменным

Бывают случаи, когда мы хотим убедиться в правильности внутреннего состояния экшена.
Представим, что у нас есть стандартная страница с пользовательским профилем.
Экшн запрашивает запись с необходимым id и определяет переменную экземпляра `@user`.
Как определить, что это именно та запись, которую мы искали?

В данном случае нас интересует [_доступ к внутренней переменной_](/guides/actions/exposures).
Он используется для передачи данных между экшеном и соответствующим представлением.
Когда мы используем `expose :user`, Hanami создает метод доступа (`#user`) и мы легко можем проверить правильность значения в переменной.

```ruby
# apps/web/controllers/users/show.rb
module Web::Controllers::Users
  class Show
    include Web::Action
    expose :user, :foo

    def call(params)
      @user = UserRepository.new.find(params[:id])
      @foo  = 'bar'
    end
  end
end
```

Мы использовали две _выстваленные внутренние переменные_: `:user` и `:foo`, проверим их значения.

```ruby
# spec/web/controllers/users/show_spec.rb
require 'spec_helper'
require_relative '../../../../apps/web/controllers/users/show'

describe Web::Controllers::Users::Show do
  before do
    @user = UserRepository.new.create(name: 'Luca')
  end

  let(:action)  { Web::Controllers::Users::Show.new }

  it "is successful" do
    response = action.call(id: @user.id)

    response[0].must_equal 200

    action.user.must_equal @user
    action.exposures.must_equal({user: @user, foo: 'bar'})
  end
end
```

<p class="notice">
Внутреннее состояние экшенов может быть проверено при помощи <em>expose</em>.
</p>

### Внедрение зависимостей

Во время модульного тестирования мы можем захотеть использовать заглушки(mocks), чтобы сделать тесты быстрее или избежать внешних вызовов к базам данным, файловой системе или удаленным сервисам.
Так как мы можем на время тестов отделить экшены от системы, то нам не потребуются антипаттерны из области тестирования (такие как `any_instance_of` или `UserRepository.new.stub(:find)`).
Вместо этого мы применим технику _внедрения зависимостей_.

Перепишем предыдущий тест так, чтобы он не использовал настоящую базу данных.
Мы используем RSpec для этого примера. В нем заглушки будут выглядеть нагляднее.

```ruby
# spec/web/controllers/users/show_spec.rb
require 'spec_helper'
require_relative '../../../../apps/web/controllers/users/show'

RSpec.describe Web::Controllers::Users::Show do
  let(:action)     { Web::Controllers::Users::Show.new(repository: repository) }
  let(:user)       { User.new(id: 23, name: 'Luca') }
  let(:repository) { double('repository', find: user) }

  it "is successful" do
    response = action.call(id: user.id)

    expect(response[0]).to      eq 200
    expect(action.user).to      eq user
    expect(action.exposures).to eq({user: user})
  end
end
```

Мы внедрили зависимость хранилища, которая на самом деле является заглушкой.
Вот как необходимо изменить экшн.

```ruby
# apps/web/controllers/users/show.rb
module Web::Controllers::Users
  class Show
    include Web::Action
    expose :user

    def initialize(repository: UserRepository.new)
      @repository = repository
    end

    def call(params)
      @user = @repository.find(params[:id])
    end
  end
end
```

<p class="warning">
Будьте осторожны с использованием double в модульных тестах. Тщательно проверяйте применимость заглушек.
</p>

## Тестирование запросов

Модульные тесты это отличный способ убедиться, что низкоуровневые интерфейсы работают так, как это ожидается.
Мы рекомендуем использовать их вместе с интеграционными тестами.

В случае веб-приложений Hanami мы можем написать приемочные тесты с Capybara, но что мы будем делать если наше приложение является HTTP API?

Необходимый нам инструмент называется `rack-test`.

Представим, что у нас есть API приложение, расположенное в `/api/v1` внутри `Hanami::Container`.

```ruby
# config/environment.rb
# ...
Hanami::Container.configure do
  mount ApiV1::Application, at: '/api/v1'
  mount Web::Application,   at: '/'
end
```

У него есть экшн.

```ruby
# apps/api_v1/controllers/users/show.rb
module ApiV1::Controllers::Users
  class Show
    include ApiV1::Action
    accept :json

    def call(params)
      user = UserRepository.new.find(params[:id])
      self.body = JSON.generate(user.to_h)
    end
  end
end
```

В этом случае нам стоит меньше беспокоиться о внутреннем состоянии экшена и больше о выходным значениях.
Вот почему мы не сделали `user` доступным извне.

```ruby
# spec/api_v1/requests/users_spec.rb
require 'spec_helper'

describe "API V1 users" do
  include Rack::Test::Methods

  before do
    @user = UserRepository.new.create(name: 'Luca')
  end

  # app необходим для Rack::Test
  def app
    Hanami.app
  end

  it "is successful" do
    get "/api/v1/users/#{ @user.id }"

    last_response.must_be :ok?
    last_response.body.must_equal(JSON.generate(@user.to_h))
  end
end
```

<p class="notice">
Избегайте <em>double</em> когда пишете интеграционные тесты, так как они должны проверять корректность выполнения всего приложения.
</p>
