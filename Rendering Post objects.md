# Отображаем Post объекты

До сих пор на главной странице ничего не отображалось. Теперь когда мы разработали систему авторизации и создали бекендную часть для модели `Post` можно позволить нашим пользователям как-то взаимодействовать с ней. Мы осуществим это, создав службу, которая позволяет получать и создавать `Post`ы, а также контроллеры и директивы для отображения данных.

## Модуль для постов

Давайте определим модуль для постов.

Создайте файл в каталоге `static/javascripts/posts` с названием `posts.module.js` и добавьте в него следующий код:

```javascript
(function () {
  'use strict';

  angular
    .module('thinkster.posts', [
      'thinkster.posts.controllers',
      'thinkster.posts.directives',
      'thinkster.posts.services'
    ]);

  angular
    .module('thinkster.posts.controllers', []);

  angular
    .module('thinkster.posts.directives', ['ngDialog']);

  angular
    .module('thinkster.posts.services', []);
})();
```

Не забудьте добавить `thinkster.posts` как зависимость для `thinkster` в файл `thinkster.js`:

```javascript
angular
  .module('thinkster', [
    'thinkster.config',
    'thinkster.routes',
    'thinkster.authentication',
    'thinkster.layout',
    'thinkster.posts'
  ]);
```

Рассматривая этот модуль отметим следующее.

Во-первых, мы создали в нём модуль с названием `thinkster.posts.directives`. Как Вы наверное догадались это означает, что мы введём понятие директив для нашего приложения в этом разделе.

Во-вторых, модулю `thinkster.posts.directives` требуется модуль `ngDialog`. `ngDialog` добавлен в шаблонный проект и используется для отображения модальных окон. Мы будем использовать модальные окна в слудющем разделе, когда будем писать код для создания новых постов.

Добавьте этот файл в `javascripts.html`:

```html
<script type="text/javascript" src="{% static 'javascripts/posts/posts.module.js' %}"></script>
```

## Создаём службу Posts

Прежде, чем мы сможем что-либо отобразить, мы должны передать данные от сервера к клиенту.

Создайте файл в каталоге `static/javascripts/posts/services/` с названием `posts.service.js` и добавьте следующий код:

```javascript
/**
* Posts
* @namespace thinkster.posts.services
*/
(function () {
  'use strict';

  angular
    .module('thinkster.posts.services')
    .factory('Posts', Posts);

  Posts.$inject = ['$http'];

  /**
  * @namespace Posts
  * @returns {Factory}
  */
  function Posts($http) {
    var Posts = {
      all: all,
      create: create,
      get: get
    };

    return Posts;

    ////////////////////

    /**
    * @name all
    * @desc Get all Posts
    * @returns {Promise}
    * @memberOf thinkster.posts.services.Posts
    */
    function all() {
      return $http.get('/api/v1/posts/');
    }


    /**
    * @name create
    * @desc Create a new Post
    * @param {string} content The content of the new Post
    * @returns {Promise}
    * @memberOf thinkster.posts.services.Posts
    */
    function create(content) {
      return $http.post('/api/v1/posts/', {
        content: content
      });
    }

    /**
     * @name get
     * @desc Get the Posts of a given user
     * @param {string} username The username to get Posts for
     * @returns {Promise}
     * @memberOf thinkster.posts.services.Posts
     */
    function get(username) {
      return $http.get('/api/v1/accounts/' + username + '/posts/');
    }
  }
})();
```

Добавьте этот файл в `javascripts.html`:

```html
<script type="text/javascript" src="{% static 'javascripts/posts/services/posts.service.js' %}"></script>
```

Этот код должен быть Вам знаком. Он очень похож на службы, которые мы создавали ранее.

Служба `Posts` содержит только два метода: `all` и `create`.

На главной странице мы будем использовать `Posts.all()` для получения списка объектов, которые мы хотим отобразить. Мы будем использовать `Posts.create()`, чтобы позволить пользователям добавлять свои собственные посты.

## Создаём интерфейс для главной страницы

Создайте `static/templates/layout/index.html` со следующим содержимым:

```html
<posts posts="vm.posts" ng-show="vm.posts && vm.posts.length"></posts>
```

Мы добавим в него кое-что ещё, но совсем не много. Большая часть необходимого нам HTML кода будет находится в шаблоне, который мы создадим для директивы `<posts>` ниже.

## Создаём службу Snackbar

В шаблонный проект для этого учебного пособия, мы добавили SnackbarJS. SnackbarJS - это небольшая JavaScript, позволяющая легко отображать snackbars (информационные окна в нижней части экрана - прим. переводчика) (понятие из Google Material Design). Здесь мы создадим службу для добавления этого функционала в наше AngularJS приложение.

Откройте `static/javascripts/utils/services/snackbar.service.js` и добавьте следующий код:

```javascript
/**
* Snackbar
* @namespace thinkster.utils.services
*/
(function ($, _) {
  'use strict';

  angular
    .module('thinkster.utils.services')
    .factory('Snackbar', Snackbar);

  /**
  * @namespace Snackbar
  */
  function Snackbar() {
    /**
    * @name Snackbar
    * @desc The factory to be returned
    */
    var Snackbar = {
      error: error,
      show: show
    };

    return Snackbar;

    ////////////////////

    /**
    * @name _snackbar
    * @desc Display a snackbar
    * @param {string} content The content of the snackbar
    * @param {Object} options Options for displaying the snackbar
    */
    function _snackbar(content, options) {
      options = _.extend({ timeout: 3000 }, options);
      options.content = content;

      $.snackbar(options);
    }


    /**
    * @name error
    * @desc Display an error snackbar
    * @param {string} content The content of the snackbar
    * @param {Object} options Options for displaying the snackbar
    * @memberOf thinkster.utils.services.Snackbar
    */
    function error(content, options) {
      _snackbar('Error: ' + content, options);
    }


    /**
    * @name show
    * @desc Display a standard snackbar
    * @param {string} content The content of the snackbar
    * @param {Object} options Options for displaying the snackbar
    * @memberOf thinkster.utils.services.Snackbar
    */
    function show(content, options) {
      _snackbar(content, options);
    }
  }
})($, _);
```

