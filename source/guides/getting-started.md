---
title: Guides - Getting Started
---

# Getting Started

<div id="getting-started-lead">
  <p>
  Привет. Если ты читаешь эту страницу, вероятно, ты хочешь узнать о Hanami больше.
  Это отлично, поздравляю! Если ты ищешь новые способы строить поддерживаемые, безопасные, быстрые и тестируемые приложения, ты в хороших руках.
  </p>

  <p>
    <strong>Hanami создан для таких же как ты.</strong>
  </p>

  <p>
    Стоит предупредить тебя, независимо от того, совсем ты новичок, или опытный разработчик <strong>процесс обучения будет сложным</strong>.
    После 10 лет, в течение которых у тебя сложился определенный взгляд на разработку, возможно, будет сложно сломать некоторые свои привычки. Изменения это всегда вызов самому себе.
  </p>

  <p>
    Иногда будет казаться, что какие-то фичи выглядят не особенно здраво, но не всегда дело в твоих взглядах.
    Это может быть делом привычки, ошибкой в проектировании или даже багом.
  </p>

  <p>
    Я и остальное сообщество из лучших побуждений стараемся улучшать Hanami каждый день.
  </p>

  <p>
    В этом руководстве мы создадим свой первый проект в Hanami, сделаем простое веб приложение 'книжная полка'.
    Мы коснемся всех основных компонентов фреймворка и покроем все написанное тестами.
  </p>

  <p>
    <strong> Если столкнешься с какими-то сложностями или просто запутаешься, не сдавайся, заходи в наш <a href="http://chat.hanamirb.org">чат</a> и задавай вопросы.</strong>
    Там всегда найдется кто-то, кто будет рад помочь.
  </p>

  <p>
    Развлекайся,<br>
    Luca Guidi<br>
    <em>Создатель Hanami</em>
  </p>
</div>

<br>
<hr>

## Предисловие

Перед тем, как мы начнем, уточним некоторые вещи. Для начала, предположим, что необходимы некоторые базовые знания о разработке веб приложений.

Ты должен быть знаком с некоторыми вещами вроде [Bundler](http://bundler.io), [Rake](http://rake.rubyforge.org), уметь работать в терминале и строить приложения с использованием паттерна [Model, View, Controller](https://ru.wikipedia.org/wiki/Model-View-Controller).

Позднее в руководстве мы будем использовать базу данных [SQLite](https://sqlite.org/) [Postgresql](https://postgrespro.ru/docs/postgresql/9.6/index.html).
Чтобы идти дальше, тебе потребуется работающая версия Ruby 2.3 или выше и SQLite 3+.

## Создадим новый Hanami проект

Чтобы создать проект в Hanami, нас сначала нужно установить gem Hanami через Rubygems.
Затем мы сможем использовать консольную команду `hanami new`, чтобы сгенерировать новый проект:

```
% gem install hanami
% hanami new bookshelf
```

  По умолчанию, проект будет настроен для использования SQLite. Для настоящих проектов вы можете указать другой движок базы данных, например, Postgres:

<code> 
  % hanami new bookshelf --database=postgres
 </code>

Это создаст новую папку `bookshelf` в той, из которой запускали команду. Посмотрите, что она будет содержать:

```
% cd bookshelf
% tree -L 1
.
├── Gemfile
├── Rakefile
├── apps
├── config
├── config.ru
├── db
├── lib
├── public
└── spec

6 directories, 3 files
```

Вот что, как минимум, стоит об этом знать:

* `Gemfile` определяет наши Rubygems зависимости (используя Bundler).
* `Rakefile` описывает наши Rake задачи.
* `apps` содержит  одно или несколько Rack совместимых приложений.
  Здесь мы можем найти первое сгенерированное Hanami приложение, называющееся `Web`.
  Там мы найдем наши контроллеры, вьюхи, маршруты и шаблоны.
* `config` содержит (внезапно!) конфигурационные файлы.
* `config.ru` (rack up) для Rack серверов.
* `db` содержит нашу схему базы данных и миграции.
* `lib` содержит нашу бизнес логику и модель предметной области, включая сущности и репозитории.
* `public` будет содержать скомпилированные ассеты и статические файлы.
* `spec` содержит наши тесты.

Двинемся дальше и установим указанные в Gemfile зависимости с помощью Bundler; затем запустим сервер в режиме разработки:

```
% bundle install
% bundle exec hanami server
```

И... встречаем твой первый Hanami проект по адресу [http://localhost:2300](http://localhost:2300)! Мы должны увидеть в браузере примерно такую картину:

<p><img src="/images/welcome-page.png" alt="Hanami welcome page" class="img-responsive"></p>

## Архитектура Hanami

Архитектура Hanami **позволяет содержать несколько Hanami (и Rack) приложений в одном процессе Ruby**.

Эти приложения находятся в каталоге `apps/`. Каждое из них может быть компонентом твоего продукта, таким как пользовательский интерфейс, панель управления, дашборд или, например, HTTP API..

Все это части _механизма доставки_ бизнес логики, живущей в папке `lib/`. Это место, где описаны наши модели предметной области, их взаимодействие друг с другом, составляющие **функциональные возможности** предоставляемые наши продуктом.

На архитектуру Hanami сильно повлияли идеи [Чистой архитектуры](https://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html) дяди Боба.

## Пишем наш первый тест

Стартовый экран приложения, который мы видели в браузере, это страница, которая показывается по умолчанию, пока мы не определили ни одного маршрута.

Hanami поощряет [Behavior Driven Development](https://en.wikipedia.org/wiki/Behavior-driven_development) (BDD) как способ разработки веб приложений. Перед созданием нашей первой страницы, мы сначала напишем высокоуровневый тест, описывающий ее работу:

```ruby
# spec/web/features/visit_home_spec.rb
require 'features_helper'

describe 'Visit home' do
  it 'is successful' do
    visit '/'

    page.body.must_include('Bookshelf')
  end
end
```

Обратите внимание, что несмотря на то, что Hanami из коробки поддерживает Разработку Через Поведение (BDD), **вас не принуждают пользоваться каким-то особенным фреймворком для тестирования** -- также не требуется какой-то особенной интеграции или библиотек.

Мы начнем с [Minitest](https://github.com/seattlerb/minitest) (который по умолчанию), но также мы можем использовать [RSpec](http://rspec.info) если создадим проект с опцией `--test=rspec.`
Hanami будет в этом случае генерировать хелперы и шаблоны файлов для него.

### Выполняем требования

У нас уже есть тест и мы можем видеть, как он падает:

```
% rake test
Run options: --seed 44759

# Running:

F

Finished in 0.018611s, 53.7305 runs/s, 53.7305 assertions/s.

  1) Failure:
Homepage#test_0001_is successful [/Users/hanami/bookshelf/spec/web/features/visit_home_spec.rb:6]:
Expected "<!DOCTYPE html>\n<html>\n  <head>\n    <title>Not Found</title>\n  </head>\n  <body>\n    <h1>Not Found</h1>\n  </body>\n</html>\n" to include "Bookshelf".

1 runs, 1 assertions, 1 failures, 0 errors, 0 skips
```

Пора сделать, чтобы он проходил. Напишем несколько строк кода, требуемого для успешного прохождения теста, шаг за шагом.

Первое, что нам нужно добавить это маршрут:

```ruby
# apps/web/config/routes.rb
root to: 'home#index'
```

Мы перенаправляем корневой (root) URL нашего приложения в экшн `index` контроллера `home` (смотри [routing guide](/guides/routing/overview) для более подробного объяснения). Теперь мы можем создать сам экшн `index`.

```ruby
# apps/web/controllers/home/index.rb
module Web::Controllers::Home
  class Index
    include Web::Action

    def call(params)
    end
  end
end
```

Это пустой экшн и в нем не содержится никакой логики. Каждый экшн имеет соответствующий вид, который представляет объект Ruby, который нужно отдать в ответ на запрос, прилетевший в экшн.

```ruby
# apps/web/views/home/index.rb
module Web::Views::Home
  class Index
    include Web::View
  end
end
```

...который, в свою очередь, тоже пуст и не делает ничего кроме рендеринга шаблона. По умолчанию это файл `erb`, но никто не запретит вам использовать `slim` или `haml`. Его нам придется поправить, чтобы тест прошел. От нас требуется лишь добавить заголовок Bookshelf.

```erb
# apps/web/templates/home/index.html.erb
<h1>Bookshelf</h1>
```

Сохраним изменения, запустим тесты еще раз и на этот раз должны пройти. Великолепно!

```shell
Run options: --seed 19286

# Running:

.

Finished in 0.011854s, 84.3600 runs/s, 168.7200 assertions/s.

1 runs, 2 assertions, 0 failures, 0 errors, 0 skips
```

## Генерируем новые экшены

Давайте же воспользуемся тем, что узнали о главных компонентах Hanami чтобы добавить новый экшн. Проект Bookshelf предназначен для учета книг.

Мы будем хранить информацию о книгах в базе данных и дадим пользователю возможность управлять ей. Первым шагом будет показ списка всех книг, хранящихся в системе.

Опишем функциональность, к которой мы стремимся, с помощью фич-теста:

```ruby
# spec/web/features/list_books_spec.rb
require 'features_helper'

describe 'List books' do
  it 'displays each book on the page' do
    visit '/books'

    within '#books' do
      assert page.has_css?('.book', count: 2), 'Expected to find 2 books'
    end
  end
end
```

Тест достаточно прост и падает, так как адрес `/books` пока что не распознается приложением. Создадим экшн контроллера, чтобы исправить это.

### Генераторы Hanami

Hanami поставляется с некоторыми **генераторами** чтобы писать меньше шаблонного кода, обрамляющего новую функциональность.
Наберите в терминале:

```
% bundle exec hanami generate action web books#index
```

Эта команда должна сгенерировать новый экшн _index_  в контроллере _books_ приложения _web_.
Что даст нам пустой экшн, вид и шаблон, а также добавит маршрут в `apps/web/config/routes.rb`:

```ruby
get '/books', to: 'books#index'
```

В ZSH ты можешь получить ошибку `zsh: no matches found: books#index`. В этом случае, попробуй другой синтаксис:
```
% hanami generate action web books/index
```

Теперь, чтобы тест прошел, нам нужно лишь сделать шаблон в файле `apps/web/templates/books/index.html.erb` похожим на:

```html
<h1>Bookshelf</h1>
<h2>All books</h2>

<div id="books">
  <div class="book">
    <h3>Patterns of Enterprise Application Architecture</h3>
    <p>by <strong>Martin Fowler</strong></p>
  </div>

  <div class="book">
    <h3>Test Driven Development</h3>
    <p>by <strong>Kent Beck</strong></p>
  </div>
</div>
```

Сохрани изменения и увидишь, что тесты проходят!

Терминология контроллеров и экшнов может сбивать с толку, так что стоит прояснить: экшены составляют основу приложения Hanami; контроллеры же просто являются модулями, объединяющими несколько экшнов.
В общем, несмотря на _концептуальное_ присутствие "контроллеров" в приложении, на практике мы будем работать только с экшнами.

Мы использовали генератор чтобы создать новую точку входа в приложение. Но стоит обратить внимание на то, что наш новый шаблон содержит тот же `<h1>` что и в `home/index.html.erb`. Давайте это исправим.

### Макеты

Для избежания повторения одних и тех же строк от шаблона к шаблону, мы можем использовать макет. Откроем файл `apps/web/templates/application.html.erb` и сделаем похожим на это:

```rhtml
<!DOCTYPE HTML>
<html>
  <head>
    <title>Bookshelf</title>
  </head>
  <body>
    <h1>Bookshelf</h1>
    <%= yield %>
  </body>
</html>
```

Теперь ты можешь убрать дублирующиеся строки из остальных шаблонов.

**Макет** похож на любой другой шаблон, но он используется для оборачивания стандартных шаблонов. Ключевое слово `yield` заменяется содержимым обычного шаблона. Это отличное место для повторяющихся элементов вроде шапки, футера или меню.

### Миграции для изменения схемы Базы Данных

Жестко зашитые в шаблон книги это надувательство, если быть честными. Пора добавить живых данных в приложение.

Во-первых, нам нужна таблица в базе данных, для хранения данных о книгах. Мы можем использовать **миграцию**, чтобы ее создать. Воспользуемся генератором для создания  пустой миграции:

```
% bundle exec hanami generate migration create_books
```

Это даст нам файл c именем вроде `db/migrations/20161115110038_create_books.rb`, отредактируем его:

```ruby
Hanami::Model.migration do
  change do
    create_table :books do
      primary_key :id

      column :title,  String, null: false
      column :author, String, null: false

      column :created_at, DateTime, null: false
      column :updated_at, DateTime, null: false
    end
  end
end
```

Hanami предоставляет специальный язык для описания изменений в схеме базы данных. Ты можешь узнать о том, как им пользоваться из [руководства по миграциям](/guides/migrations/overview).

В этом случае, мы определяем новую таблицу с колонками для каждого атрибута нашей сущности.
Давайте подготовим базу данных:

```
% bundle exec hanami db prepare
```

## Выражаем наши данные в виде Сущностей

Мы храним книги в нашей базе данных и показываем их на странице. Для этого, нам нужен способ читать и писать в БД. Представляем сущности и репозитории:

*  **сущность** это объект из предметной области (типа `Book`) уникально определяемый по его идентификатору.
* **репозиторий** находится между сущностями и слоем, обеспечивающим непрерывность их существования.

Сущности абсолютно независимы от базы данных. Это делает их **легкими** и **просто тестируемыми**.

По этой причине нам нужен репозиторий для сохранения данных от которых `Book` зависит.
Подробнее о сущностях и репозиториях в [руководству по моделям](/guides/models/overview).

Hanami предоставляет генератор для моделей, так что давайте создадим сущность `Book` и соответствующий репозиторий:

```
% bundle exec hanami generate model book
create  lib/bookshelf/entities/book.rb
create  lib/bookshelf/repositories/book_repository.rb
create  spec/bookshelf/entities/book_spec.rb
create  spec/bookshelf/repositories/book_repository_spec.rb
```

Генератор дает нам сущность, репозиторий и сопутствующие файлы для тестов.

### Работаем с Сущностями

Сущности это что-то близкое по сути к простым объектам Ruby. Мы должны сфокусироваться на поведении, которого мы от них хотим и уже потом на том, как их сохранять.

Прямо сейчас нам нужен простой класс сущности:

```ruby
# lib/bookshelf/entities/book.rb
class Book < Hanami::Entity
end
```

Этот класс сгенерирует геттеры и сеттеры для каждого атрибута, передаваемого как параметр при инициализации. Мы можем в этом убедиться написав модульный тест:

```ruby
# spec/bookshelf/entities/book_spec.rb
require 'spec_helper'

describe Book do
  it 'can be initialised with attributes' do
    book = Book.new(title: 'Refactoring')
    book.title.must_equal 'Refactoring'
  end
end
```

### Использование Репозиториев

Теперь мы готовы поиграть с репозиторием. С помощью команды Hanami `console`, мы запустим IRb в контексте нашего приложения, что позволит нам использовать существующие объекты:

```
% bundle exec hanami console
>> repository = BookRepository.new
=> => #<BookRepository:0x007f9ab61fbb40 ...>
>> repository.all
=> []
>> book = repository.create(title: 'TDD', author: 'Kent Beck')
=> #<Book:0x007f9ab61c23b8 @attributes={:id=>1, :title=>"TDD", :author=>"Kent Beck", :created_at=>2016-11-15 11:11:38 UTC, :updated_at=>2016-11-15 11:11:38 UTC}>
>> repository.find(book.id)
=> #<Book:0x007f9ab6181610 @attributes={:id=>1, :title=>"TDD", :author=>"Kent Beck", :created_at=>2016-11-15 11:11:38 UTC, :updated_at=>2016-11-15 11:11:38 UTC}>
```

Репозитории Hanami имеют методы для загрузки как одной, так и нескольких сущностей из БД; а также для создания и обновления существующих. Также можно определить в репозитории новые методы для собственных запросов.

В итоге, мы видели как Hanami использует сущности и репозитории для моделирования наших данных. Сущности отражают поведение, а репозитории используются для связи сущностей и хранилища данных. Мы можем использовать миграции, чтобы применить изменения к схеме базы данных.

### Показываем динамические данные

С нашим новым опытом моделирования данных, мы можем заставить страницу со списком книг показывать изменяющиеся данные. Приспособим для этого написанный ранее тест:

```ruby
# spec/web/features/list_books_spec.rb
require 'features_helper'

describe 'List books' do
  let(:repository) { BookRepository.new }
  before do
    repository.clear

    repository.create(title: 'PoEAA', author: 'Martin Fowler')
    repository.create(title: 'TDD',   author: 'Kent Beck')
  end

  it 'displays each book on the page' do
    visit '/books'

    within '#books' do
      assert page.has_css?('.book', count: 2), 'Expected to find 2 books'
    end
  end
end
```

Мы создали требуемые записи в тесте и затем заявили, что число классов книг на странице им соответствует. Когда мы запустим тесты заново, скорее всего увидим ошибку, связанную с базой данных -- помните, что мы уже мигрировали _development_ базу, но еще не провели миграцию базы _test_.

```
% HANAMI_ENV=test bundle exec hanami db prepare
```

Сейчас мы можем изменить шаблон и убрать статический HTML. Наши вьюхи должны пройтись по всем доступным записям и отрендерить их. Напишем тест, требующий такого изменения для вида:

```ruby
# spec/web/views/books/index_spec.rb
require 'spec_helper'
require_relative '../../../../apps/web/views/books/index'

describe Web::Views::Books::Index do
  let(:exposures) { Hash[books: []] }
  let(:template)  { Hanami::View::Template.new('apps/web/templates/books/index.html.erb') }
  let(:view)      { Web::Views::Books::Index.new(template, exposures) }
  let(:rendered)  { view.render }

  it 'exposes #books' do
    view.books.must_equal exposures.fetch(:books)
  end

  describe 'when there are no books' do
    it 'shows a placeholder message' do
      rendered.must_include('<p class="placeholder">There are no books yet.</p>')
    end
  end

  describe 'when there are books' do
    let(:book1)     { Book.new(title: 'Refactoring', author: 'Martin Fowler') }
    let(:book2)     { Book.new(title: 'Domain Driven Design', author: 'Eric Evans') }
    let(:exposures) { Hash[books: [book1, book2]] }

    it 'lists them all' do
      rendered.scan(/class="book"/).count.must_equal 2
      rendered.must_include('Refactoring')
      rendered.must_include('Domain Driven Design')
    end

    it 'hides the placeholder message' do
      rendered.wont_include('<p class="placeholder">There are no books yet.</p>')
    end
  end
end
```

Мы указали, что страница индекса будет показывать сообщение когда нет книг и нечего показывать; когда же книги есть, то будет показывать их список. Обратите внимание, что рендеринг вьюхи с данными относительно прямолинеен. Hanami спроектирован из простых объектов с минимальными интерфейсами, что позволяет их легко протестировать по отдельности, к тому же они отлично работают вместе.

Давайте перепишем наш шаблон, чтобы воплотить задуманное:

```erb
# apps/web/templates/books/index.html.erb
<h2>All books</h2>

<% if books.any? %>
  <div id="books">
    <% books.each do |book| %>
      <div class="book">
        <h2><%= book.title %></h2>
        <p><%= book.author %></p>
      </div>
    <% end %>
  </div>
<% else %>
  <p class="placeholder">There are no books yet.</p>
<% end %>
```

Если же мы прогоним наш функциональный тест снова, он упадет, так как наш экшн контроллера все еще  не [_выставляет напоказ (expose)_](/guides/actions/exposures) книги для нашей вьюхи. Мы можем написать тест для этого дела:

```ruby
# spec/web/controllers/books/index_spec.rb
require 'spec_helper'
require_relative '../../../../apps/web/controllers/books/index'

describe Web::Controllers::Books::Index do
  let(:action) { Web::Controllers::Books::Index.new }
  let(:params) { Hash[] }
  let(:repository) { BookRepository.new }

  before do
    repository.clear

    @book = repository.create(title: 'TDD', author: 'Kent Beck')
  end

  it 'is successful' do
    response = action.call(params)
    response[0].must_equal 200
  end

  it 'exposes all books' do
    action.call(params)
    action.exposures[:books].must_equal [@book]
  end
end
```

Написание тестов для экшенов обычно имеет две стороны: ты делаешь утверждение относительно объекта ответа, который представляет собой Rack-совместимый массив из статуса, заголовков и контента; или про то, что из экшна видны данные после того, как мы его вызвали. Сейчас мы указали, что экшн показывает переменную переменную `:books`, что мы и сделаем:

```ruby
# apps/web/controllers/books/index.rb
module Web::Controllers::Books
  class Index
    include Web::Action

    expose :books

    def call(params)
      @books = BookRepository.new.all
    end
  end
end
```

Используя метод класса экшна `expose`, мы можем выставить напоказ содержимое переменной экземпляра `@books`, что делает ее доступной во вьюхе. Этого достаточно, чтобы тесты опять проходили!

```shell
% bundle exec rake
Run options: --seed 59133

# Running:

.........

Finished in 0.042065s, 213.9543 runs/s, 380.3633 assertions/s.

6 runs, 7 assertions, 0 failures, 0 errors, 0 skips
```

## Строение форм для создания записей

Остается только создать возможность добавлять новые книги в систему. План прост: мы сделаем страницу с формой, куда можно ввести подробности.

Когда пользователь отправит форму, мы построим новую сущность, сохраним ее и перенаправим пользователя на страницу со списком книг. Вырази историю в тесте:

```ruby
# spec/web/features/add_book_spec.rb
require 'features_helper'

describe 'Add a book' do
  after do
    BookRepository.new.clear
  end

  it 'can create a new book' do
    visit '/books/new'

    within 'form#book-form' do
      fill_in 'Title',  with: 'New book'
      fill_in 'Author', with: 'Some author'

      click_button 'Create'
    end

    current_path.must_equal('/books')
    assert page.has_content?('New book')
  end
end
```

### Закладываем основы Форм

На этот момент, нам должно быть известно, как работают экшены, виды и шаблоны.

Мы немного ускорим процесс, и сразу перейдем к интересной части. Сначала создадим новый экшн для страницы `New Book`:

```
% bundle exec hanami generate action web books#new
```

Это добавит новый маршрут в приложение:

```ruby
# apps/web/config/routes.rb
get '/books/new', to: 'books#new'
```

Следующий интересный момент связан с шаблоном, так как мы будем использовать встроенный в Hanami конструктор форм, для генерации HTML  формы для сущности `Book`:

### Использование хелперов форм

Вослользуемся [хелперами форм](/guides/helpers/forms) и создадим одну `apps/web/templates/books/new.html.erb`:

```erb
# apps/web/templates/books/new.html.erb
<h2>Add book</h2>

<%=
  form_for :book, '/books' do
    div class: 'input' do
      label      :title
      text_field :title
    end

    div class: 'input' do
      label      :author
      text_field :author
    end

    div class: 'controls' do
      submit 'Create Book'
    end
  end
%>
```

Мы добавляем тэги `<label>` для каждого поля формы, и оборачиваем каждое поле в контейнер `<div>` иcпользуя Hanami [хелпер HTML](/guides/helpers/html5).

### Отправка наших Форм

Чтобы отправить форму, нам нужен еще один экшн. Давайте создадим экшн `Books::Create`:

```
% bundle exec hanami generate action web books#create --method=post
```

Это добавит новый маршрут в приложение:

```ruby
# apps/web/config/routes.rb
post '/books', to: 'books#create'
```

### Воплощаем экшн Create

Наш экшн `books#create` нуждается в двух вещах. Выразим их в юнит-тестах:

```ruby
# spec/web/controllers/books/create_spec.rb
require 'spec_helper'
require_relative '../../../../apps/web/controllers/books/create'

describe Web::Controllers::Books::Create do
  let(:action) { Web::Controllers::Books::Create.new }
  let(:params) { Hash[book: { title: 'Confident Ruby', author: 'Avdi Grimm' }] }

  before do
    BookRepository.new.clear
  end

  it 'creates a new book' do
    action.call(params)

    action.book.id.wont_be_nil
    action.book.title.must_equal params[:book][:title]
  end

  it 'redirects the user to the books listing' do
    response = action.call(params)

    response[0].must_equal 302
    response[1]['Location'].must_equal '/books'
  end
end
```

Сделать, чтобы они прошли достаточно просто. Мы уже видели, как мы можем писать сущности в базу данных, и мы можем использовать `redirect_to` для реализации перенаправления:

```ruby
# apps/web/controllers/books/create.rb
module Web::Controllers::Books
  class Create
    include Web::Action

    expose :book

    def call(params)
      @book = BookRepository.new.create(params[:book])

      redirect_to '/books'
    end
  end
end
```

Этой минималистичной реализации уже должно быть достаточно, чтобы наши тесты снова проходили успешно.

```shell
% bundle exec rake
Run options: --seed 63592

# Running:

...............

Finished in 0.081961s, 183.0142 runs/s, 305.0236 assertions/s.

12 runs, 14 assertions, 0 failures, 0 errors, 2 skips
```

С чем и поздравляем!

### Защитим формы валидациями

Придержи коней! Нужно немного терпения, чтобы сделать еще и защиту от дурака. Представьте, что случится, когда кто-то отправит форму не заполнив поля?

Мы можем заполнить базу данных неверными данными или увидеть ошибку про нарушение целостности данных. Нам точно нужен способ держать невалидные данные подальше от нашей системы!

Чтобы выразить проверки в тесте, нам надо задуматься: что должно _случиться_ если проверка провалится? Разумно будет перерендерить форму `bookshelf#new`, так мы дадим пользователю еще один шанс заполнить ее правильно. Опишем это поведение в юнит-тестах:

```ruby
# spec/web/controllers/books/create_spec.rb
require 'spec_helper'
require_relative '../../../../apps/web/controllers/books/create'

describe Web::Controllers::Books::Create do
  let(:action) { Web::Controllers::Books::Create.new }

  after do
    BookRepository.new.clear
  end

  describe 'with valid params' do
    let(:params) { Hash[book: { title: '1984', author: 'George Orwell' }] }

    it 'creates a new book' do
      action.call(params)
      action.book.id.wont_be_nil
    end

    it 'redirects the user to the books listing' do
      response = action.call(params)

      response[0].must_equal 302
      response[1]['Location'].must_equal '/books'
    end
  end

  describe 'with invalid params' do
    let(:params) { Hash[book: {}] }

    it 're-renders the books#new view' do
      response = action.call(params)
      response[0].must_equal 422
    end

    it 'sets errors attribute accordingly' do
      response = action.call(params)
      response[0].must_equal 422

      action.params.errors[:book][:title].must_equal  ['is missing']
      action.params.errors[:book][:author].must_equal ['is missing']
    end
  end
end
```

Теперь наши тесты описывают два альтернативных сценария: наш оригинальный успешный путь и новый сценарий, в котором проверка проваливается. Чтобы починить тесты, сделаем валидации.

Мы, конечно, могли бы поместить все правила валидации в сущность, Hanami также позволяет определять правила валидации несколько ближе к источнику пользовательского ввода, то есть прямо в экшенах. Экшены Hanami могут использовать метод класса `params` для определения допустимых значений параметров.

Этот подход позволяет одновременно: задать белый список параметров (остальные игнорируются, ради предотвращения уязвимости перед недоверенным пользовательским вводом) _и_ позволяет указать, какие значения принимаются - в этом случае, мы указали, что атрибуты книги, а именно автор и название должны быть заполнены.

С подходящими валидациями, мы можем отделить случай, в котором создание сущности и перенаправление, когда входящие параметры верные

```ruby
# apps/web/controllers/books/create.rb
module Web::Controllers::Books
  class Create
    include Web::Action

    expose :book

    params do
      required(:book).schema do
        required(:title).filled(:str?)
        required(:author).filled(:str?)
      end
    end

    def call(params)
      if params.valid?
        @book = BookRepository.new.create(params[:book])

        redirect_to '/books'
      else
        self.status = 422
      end
    end
  end
end
```

Когда параметры валидные, Книга создается и экшн перенаправляет нас на другой URL. Но что должно произойти, когда параметры неверны?

Сначала код статуса HTTP устанавливается как
[422 (Необрабатываемая сущность)](https://ru.wikipedia.org/wiki/List_of_HTTP_status_codes#422).
Затем контроль передается соответствующему виду, который должен знать, какой шаблон показывать. В нашем случае `apps/web/templates/books/new.html.erb` будет использован чтобы снова показать форму.

```ruby
# apps/web/views/books/create.rb
module Web::Views::Books
  class Create
    include Web::View
    template 'books/new'
  end
end
```

Этот подход работает, потому что конструктор форм Hanami достаточно крутой, чтобы проверить `params` в этом экшене и заполнить поля формы значениями, найденными в параметрах. Если пользователь заполнит только одно поле перед отправкой, поля предстанут в том виде, как их ввели, не заставляя пользователя снова их вводить.

Запусти тесты снова и они все должны быть успешны.

### Показываем ошибки валидации

Мало повторно ткнуть пользователя носом в форму и сказать, что что-то пошло не так, мы должны подсказать ему, чего мы от него ждем. Давайте же адаптируем форму чтобы показывать уведомление о неверный полях.

Для начала, мы ожидаем что список ошибок будет вставлен в страницу, когда `params` содержит ошибки:

```ruby
# spec/web/views/books/new_spec.rb
require 'spec_helper'
require_relative '../../../../apps/web/views/books/new'

class NewBookParams < Hanami::Action::Params
  params do
    required(:book).schema do
      required(:title).filled(:str?)
      required(:author).filled(:str?)
    end
  end
end

describe Web::Views::Books::New do
  let(:params)    { NewBookParams.new(book: {}) }
  let(:exposures) { Hash[params: params] }
  let(:template)  { Hanami::View::Template.new('apps/web/templates/books/new.html.erb') }
  let(:view)      { Web::Views::Books::New.new(template, exposures) }
  let(:rendered)  { view.render }

  it 'displays list of errors when params contains errors' do
    params.valid? # trigger validations

    rendered.must_include('There was a problem with your submission')
    rendered.must_include('Title is missing')
    rendered.must_include('Author is missing')
  end
end
```

Мы должны также обновить наш тест возможностей чтобы отразить это новое поведение:

```ruby
# spec/web/features/add_book_spec.rb
require 'features_helper'

describe 'Add a book' do
  # Spec written earlier omitted for brevity

  it 'displays list of errors when params contains errors' do
    visit '/books/new'

    within 'form#book-form' do
      click_button 'Create'
    end

    current_path.must_equal('/books')

    assert page.has_content?('There was a problem with your submission')
    assert page.has_content?('Title must be filled')
    assert page.has_content?('Author must be filled')
  end
end
```

В шаблоне мы можем перебрать все `params.errors` (если они есть) и показать дружелюбное сообщение.
Откроем `apps/web/templates/books/new.html.erb`:

```erb
<% unless params.valid? %>
  <div class="errors">
    <h3>There was a problem with your submission</h3>
    <ul>
      <% params.error_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
    </ul>
  </div>
<% end %>
```

Как видишь, мы просто захардкодили сообщение об ошибке "is required", но ты можешь проверить содержание ошибок и разнообразить сообщения для указания, какие именно валидации провалились. Это будет усовершенствовано в ближайшем будущем.

```shell
% bundle exec rake
Run options: --seed 59940

# Running:

..................

Finished in 0.078112s, 230.4372 runs/s, 473.6765 assertions/s.

15 runs, 27 assertions, 0 failures, 0 errors, 1 skips
```

### Используем Роутер более осмысленно

Последнее что мы собираемся улучшить, это наш способ использования роутера. Откроем файл маршрутов приложения "web":

```ruby
# apps/web/config/routes.rb
post '/books',    to: 'books#create'
get '/books/new', to: 'books#new'
get '/books',     to: 'books#index'
root              to: 'home#index'
```

Hanami предлагает более удобный способ через хелпер построить эти REST-подобные маршруты, так что мы можем немного упростить роутер:

```ruby
resources :books, only: [:index, :new, :create]
root to: 'home#index'
```

Чтобы убедиться, что маршруты определены, после того, как мы внесли изменения, можно
использовать задачу командной строки `routes` и проверить результат:

```
% bundle exec hanami routes
                Name Method     Path                           Action

               books GET, HEAD  /books                         Web::Controllers::Books::Index
            new_book GET, HEAD  /books/new                     Web::Controllers::Books::New
               books POST       /books                         Web::Controllers::Books::Create
                root GET, HEAD  /                              Web::Controllers::Home::Index
```

Вывод команды `hanami routes` показывает список определенных имен вспомогательных методов (которые мы можем дополнить окончанием `_path` или `_url` и вызывать на хелпере `routes`), разрешенный HTTP метод и экшн контроллера, который должен обработать запрос.

Теперь, когда мы применили хелпер `resources`, мы можем воспользоваться методами именованых маршрутов. Помнишь, как мы делали форму с `form_for`?

```erb
<%=
  form_for :book, '/books' do
    # ...
  end
%>
```

Хардкодить пути в шаблонах не лучшая идея, особенно, когда роутер уже прекрасно знает, какой маршрут нужно указать в форме. Мы можем использовать метод хелпера `routes`, который уже доступен в наших видах и экшенах, чтобы добыть более специфичные методы хелпера:

```erb
<%=
  form_for :book, routes.books_path do
    # ...
  end
%>
```

МЫ можем сделать то же самое в `apps/web/controllers/books/create.rb`:

```ruby
redirect_to routes.books_path
```

## Резюмируем

**Поздравляем с завершением твоего первого Hanami проекта!**

Посмотрим, что мы успели натворить: мы отследили путь запроса через основные механизмы Hanami, чтобы понять, как они друг с другом связаны; мы моделировали нашу предметную область используя сущности и репозитории; мы видели решения для создания форм, управления схемой базы данных и проверки введенных пользователем данных.

Мы прошли долгий путь, но остается узнать еще много всего.
Просмотри [другие руководства](/guides), the [документацию Hanami API](http://www.rubydoc.info/gems/hanami), прочти [исходный код](https://github.com/hanami) и следи за [блогом](/blog).

**И главное, наслаждайся творчеством!**

<div class="block block-bordered-lg text-center">
  <div class="container-fluid">
    <p class="lead m-b-md">
    Присоединиться к более чем 2,000+ разработчиков.
    </p>
    <form action="http://hanamirb.us3.list-manage.com/subscribe/post" method="POST" class="form-inline">
      <input name="u" value="dcbeefa4ba1ea9ae043857005" type="hidden">
      <input name="id" value="fb3873a90f" type="hidden">
      <input name="orig-lang" value="1" type="hidden">
      <input type="email" class="form-control m-b" placeholder="Email Address" spellcheck="false" autocapitalize="off" autocorrect="off" name="MERGE0" id="MERGE0" value="">
      <button class="btn btn-primary m-b">Subscribe</button>
    </form>
    <small class="text-muted">
      Нажимая "Подписаться" я выражаю желание подписаться на рассылку Hanami.
    </small>
  </div>
</div>
