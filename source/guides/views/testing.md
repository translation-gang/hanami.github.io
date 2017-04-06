---
title: Руководство - Представления: Тестирование
---

# Тестирование

Одним из преимуществ представлений как объектов является простота их модульного тестирования.
Мы легко можем проверить, представлено ли ожидаемое содержимое в результате рендера страницы.

В следующем примере мы используем Rspec. У него есть удобный синтаксис для задания входных значений.

```ruby
# spec/web/views/books/show_spec.rb
require 'spec_helper'
require_relative '../../../../apps/web/views/books/show'

RSpec.describe Web::Views::Books::Show do
  let(:exposures) { Hash[book: double('book', price: 1.00), current_user: user] }
  let(:template)  { Hanami::View::Template.new('apps/web/templates/books/show.html.erb') }
  let(:view)      { Web::Views::Home::Another.new(template, exposures) }
  let(:rendered)  { view.render }
  let(:user)      { double('user', admin?: false) }

  describe "price" do
    it "returns formatted price" do
      expect(view.formatted_price).to eq "$1.00"
    end
  end

  describe "edit link" do
    it "doesn't show it by default" do
      expect(rendered).to_not match %(<a href="">edit</a>)
    end

    context "when admin" do
      let(:user) { double('user', admin?: true) }

      it "shows it" do
        expect(rendered).to match %(<a href="">edit</a>)
      end
    end
  end
end
```

Первая часть кода в тесте выше относится к форматированию цены на книги.
Отображение представления же проверяется в `view.formatted_price`.

Дальнейший код проверяет управление доступом: ссылка на редактирование должна быть доступна только если пользователь является администратором.
Эту часть легко протестировать. Нужно только посмотреть на вывод шаблонизатора.

<p class="notice">
  Есть два равноценных способа тестирования представлений: через методы непосредственно представлений и через анализ вывода шаблонизатора.
</p>

Проверяемый код:

```ruby
# apps/web/views/books/show.rb
module Web::Views::Books
  class Show
    include Web::View

    def formatted_price
      "$#{ format_number book.price }"
    end

    def edit_link
      if can_edit_book?
        link_to "Edit", routes.edit_book_path(id: book.id)
      end
    end

    private

    def can_edit_book?
      current_user.admin?
    end
  end
end
```
