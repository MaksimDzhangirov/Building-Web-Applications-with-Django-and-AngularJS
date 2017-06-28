# Отображаем профили пользователей

Мы уже создали в Django представления и маршруты, необходимые для отображения профиля каждого пользователя. Таким образом, мы можем перейти к созданию AngularJS службы, а затем к шаблонам и контроллерам.

> Замечание. В этом и следующем разделе мы будем называть учетные записи профилями. Для нашего клиента модель `Account` по существу преобразуется в профиль пользователя.

## Создаём модуль для профилей

Мы создадим службу и несколько контроллеров, связанных с профилями пользователей, поэтому давайте продолжим и определим необходимые нам модули.

Создайте файл `static/javascripts/profiles/profiles.module.js` со следующим содержимым:

```javascript
(function () {
  'use strict';

  angular
    .module('thinkster.profiles', [
      'thinkster.profiles.controllers',
      'thinkster.profiles.services'
    ]);

  angular
    .module('thinkster.profiles.controllers', []);

  angular
    .module('thinkster.profiles.services', []);
})();
```

Как обычно не забудьте добавить `thinkster.profiles` как зависимость в `thinkster` в `thinkster.js`:

```javascript
angular
  .module('thinkster', [
    'thinkster.config',
    'thinkster.routes',
    'thinkster.authentication',
    'thinkster.layout',
    'thinkster.posts',
    'thinkster.profiles'
  ]);
```

Включите этот файл в `javascripts.html`:

```html
<script type="text/javascript" src="{% static 'javascripts/profiles/profiles.module.js' %}"></script>
```

## Создаём Profile фабрику

