---
title: Руководство - Content Delivery Network (CDN)
---

# Ассеты

## Content Delivery Network (CDN)

Приложения Ханами могут раздавать ассеты из [Content Delivery Network](https://ru.wikipedia.org/wiki/Content_delivery_network) (CDN).
Эта функция особенно полезна в режиме _эксплуатации_, когда мы хотим ускорить раздачу ассетов.

Чтобы воспользоваться ей, потребуется указать настройки CDN.

```ruby
# apps/web/application.rb
module Web
  class Application < Hanami::Application
    # ...
    configure :production do
      scheme 'https'
      host   'bookshelf.org'
      port   443

      assets do
        # ...
        fingerprint true

        # CDN settings
        scheme 'https'
        host   '123.cloudfront.net'
        port   443
      end
    end
  end
end
```

Включив CDN один раз, все [хэлперы ассетов](/guides/helpers/assets) будут возвращать нужные **абсолютные URL**.

```erb
<%= stylesheet 'application' %>
```

```html
<link href="https://123.cloudfront.net/assets/application-9ab4d1f57027f0d40738ab8ab70aba86.css" type="text/css" rel="stylesheet">
```

## Целостность подресурсов

CDN способен значительно ускорить загрузку ваших страниц, но вместе с тем может стать причиной уязвимости в вашей системе безопасности.

Если ваш CDN будет скомпроментирован и начнет раздавать файлы JavaScript и таблиц стилей злоумышленников, то пользователи вашего приложения могут стать жертвой атак как XSS.

Для решения этой проблемы поставщики браузеров создали средство защиты, названное [проверкой целостности подресурсов](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity).

Когда эта функция включена, браузер будет сравнивать контрольные суммы загружаемых файлов CDN с полученными от вашего приложения.

### В случае CDN

Если мы хотим использовать jQuery из их CDN, то нам нужно найти контрольные суммы нужного файла `.js` и написать:


```erb
<%= javascript 'https://code.jquery.com/jquery-3.1.0.min.js', integrity: 'sha256-cCueBR6CsyA4/9szpPfrX3s49M9vUU5BgtiJj06wt/s=' %>
```

На выходе получим:

```html
<script integrity="sha256-cCueBR6CsyA4/9szpPfrX3s49M9vUU5BgtiJj06wt/s=" src="https://code.jquery.com/jquery-3.1.0.min.js" type="text/javascript" crossorigin="anonymous"></script>
```

### Локальные ассеты

Проблема безопасности актуальна не только для ассетов с CDN, но и для локальных файлов тоже.
Представим, что кто-то получил доступ к нашей файловой системе и заменил наши сценарии JavaScript своими.

Для исправления этой уязвимости Ханами **по умолчанию включает проверку целостности подресурсов в режиме эксплуатации**.
Когда мы проводим [прекомпиляцию ассетов](/guides/command-line/assets) перед развертыванием прилоежния, Ханами подсчитывает контрольные суммы всех ассетов и добавляет специальный HTML атрибут `integrity` ко всем тэгам ассетов, таким как `<script>`.

```erb
<%= javascript 'application' %>
```

```html
<script src="/assets/application-92cab02f6d2d51253880cd98d91f1d0e.js" type="text/javascript" integrity="sha256-WB2pRuy8LdgAZ0aiFxLN8DdfRjKJTc4P4xuEw31iilM=" crossorigin="anonymous"></script>
```

### Настройки

Чтобы отключить или настроить эту функцию, обратитесь к блоку `production` в конфигурации `apps/web/application.rb`

```ruby
module Web
  class Application < Hanami::Application
    configure :production do
      assets do
        # ...
        subresource_integrity :sha256
      end
    end
  end
end
```

Закомментировав или удалив эту строку мы отключим эту функцию.

Также мы можем выбрать один или несколько алгоритмов подсчета:

```ruby
subresource_integrity :sha256, :sha512
```

Для такой конфигурации, Ханами будет выводить HTML атрибут `integrity` с двумя значениями: одним для алгоритма `SHA256`, и другим для `SHA512`.

```html
<script src="/assets/application-92cab02f6d2d51253880cd98d91f1d0e.js" type="text/javascript" integrity="sha256-WB2pRuy8LdgAZ0aiFxLN8DdfRjKJTc4P4xuEw31iilM= sha512-4gegSER1uqxBvmlb/O9CJypUpRWR49SniwUjOcK2jifCRjFptwGKplFWGlGJ1yms+nSlkjpNCS/Lk9GoKI1Kew==" crossorigin="anonymous"></script>
```

**Обратите внимание**, подсчет контрольных сумм является интенсивной вычислительной операцией. Поэтому добавление каждого алгоритма в `subresource_integrity` будет увеличивать время _прекомпеляции ассетов_ перед развертыванием приложения. Мы рекомендуем использовать значение по умолчанию(`:sha256`).
