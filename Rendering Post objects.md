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

Не забудьте настроить Ваши модули. Откройте `static/javascripts/utils/utils.module.js` и добавьте следующее:

```javascript
(function () {
  'use strict';

  angular
    .module('thinkster.utils', [
      'thinkster.utils.services'
    ]);

  angular
    .module('thinkster.utils.services', []);
})();
```

Добавьте `thinksters.utils` в качестве зависимости для `thinkster` в `static/javascripts/thinkster.js`:

```javascript
angular
  .module('thinkster', [
    // ...
    'thinkster.utils',
    // ...
  ]);
```

Последний этап при создании этой службы - включение новых JavaScript файлов в `javascripts.html`:

```html
<script type="text/javascript" src="{% static 'javascripts/utils/utils.module.js' %}"></script>
<script type="text/javascript" src="{% static 'javascripts/utils/services/snackbar.service.js' %}"></script>
```

## Управляем интерфейсом главной страницы с помощью IndexController

Создайте файл в `static/javascripts/layout/controllers/` под названием `index.controller.js` и добавьте в него следующий код:

```javascript
/**
* IndexController
* @namespace thinkster.layout.controllers
*/
(function () {
  'use strict';

  angular
    .module('thinkster.layout.controllers')
    .controller('IndexController', IndexController);

  IndexController.$inject = ['$scope', 'Authentication', 'Posts', 'Snackbar'];

  /**
  * @namespace IndexController
  */
  function IndexController($scope, Authentication, Posts, Snackbar) {
    var vm = this;

    vm.isAuthenticated = Authentication.isAuthenticated();
    vm.posts = [];

    activate();

    /**
    * @name activate
    * @desc Actions to be performed when this controller is instantiated
    * @memberOf thinkster.layout.controllers.IndexController
    */
    function activate() {
      Posts.all().then(postsSuccessFn, postsErrorFn);

      $scope.$on('post.created', function (event, post) {
        vm.posts.unshift(post);
      });

      $scope.$on('post.created.error', function () {
        vm.posts.shift();
      });


      /**
      * @name postsSuccessFn
      * @desc Update posts array on view
      */
      function postsSuccessFn(data, status, headers, config) {
        vm.posts = data.data;
      }


      /**
      * @name postsErrorFn
      * @desc Show snackbar with error
      */
      function postsErrorFn(data, status, headers, config) {
        Snackbar.error(data.error);
      }
    }
  }
})();
```

Добавьте этот файл в `javascripts.html`:

```html
<script type="text/javascript" src="{% static 'javascripts/layout/controllers/index.controller.js' %}"></script>
```

Уделим внимание некоторым моментам в этом фрагменте кода.

```javascript
$scope.$on('post.created', function (event, post) {
  vm.posts.unshift(post);
});
```

Позднее, при создании новых постов, мы будем запускать событие под названием `post.created`, каждый раз, когда пользователь создаёт пост. Перехватывая это событие здесь мы можем добавить этот новый пост в начало массива `vm.posts`. Это позволит нам не выполнять дополнительные запросы к серверному API для получения обновленных данных. Вскоре мы поговорим об этом более подробно, а пока что Вам нужно знать, что мы делаем это, чтобы повысить производительность нашего приложения *с точки зрения восприятия пользователем*.

```javascript
$scope.$on('post.created.error', function () {
  vm.posts.shift();
});
```

По аналогии с предыдущим перехватчиком события, этот будет удалять пост из начала `vm.posts`, если API возвращает код статуса, соотвутствующий ошибке.

## Создаём маршрут для главной страницы

Закончив с контроллером и шаблоном, нам необходимо настроить маршрут для главной страницы.

Откройте `static/javascripts/thinkster.routes.js` и добавьте следующий маршрут:

```javascript
.when('/', {
  controller: 'IndexController',
  controllerAs: 'vm',
  templateUrl: '/static/templates/layout/index.html'
})
```

## Создаём директиву для отображения Posts

Создайте файл `static/javascripts/posts/directives/posts.directive.js` со следующим содержимым:

```javascript
/**
* Posts
* @namespace thinkster.posts.directives
*/
(function () {
  'use strict';

  angular
    .module('thinkster.posts.directives')
    .directive('posts', posts);

  /**
  * @namespace Posts
  */
  function posts() {
    /**
    * @name directive
    * @desc The directive to be returned
    * @memberOf thinkster.posts.directives.Posts
    */
    var directive = {
      controller: 'PostsController',
      controllerAs: 'vm',
      restrict: 'E',
      scope: {
        posts: '='
      },
      templateUrl: '/static/templates/posts/posts.html'
    };

    return directive;
  }
})();
```

Добавьте этот файл в `javascripts.html`:

```html
<script type="text/javascript" src="{% static 'javascripts/posts/directives/posts.directive.js' %}"></script>
```

Существует два момента, касающихся API директив, которые я хотел бы рассмотреть: `scope` и `restrict`.

```javascript
scope: {
  posts: '='
},
```

`scope` определяет область видимости этой директивы по аналогии с тем как `$scope` используется для контроллеров. Разница заключается в том, что контроллере новая область видимости создаётся неявно. Для директивы мы можем явно определить области видимости, чем мы и воспользовались здесь.

Вторая строка `posts: '='` просто означает, что мы хотим присвоить `$scope.posts` значение, которое будет передано в атрибуте `posts` в шаблоне, созданном ранее.

```javascript
restrict: 'E',
```

`restrict` указывает Angular как нам можно использовать эту директиву. В нашем случае мы присваиваем `restrict` значение `E` (что означает element), таким образом, Angular должен использовать нашу директиву только для элементов с названием `<posts></posts>`.

Другим часто используемым значением является `A` (attribute), что указывает Angular использовать директиву только для элементов с таким атрибутом. Как мы скоро увидим такое значение используется в `ngDialog`.

## Управляем директивой для постов с помощью PostsController

Директиве, которую мы только что создали, нужен контроллер с названием `PostsController`.

Создайте файл `static/javascripts/posts/controllers/posts.controller.js` со следующим содержимым:

```javascript
/**
* PostsController
* @namespace thinkster.posts.controllers
*/
(function () {
  'use strict';

  angular
    .module('thinkster.posts.controllers')
    .controller('PostsController', PostsController);

  PostsController.$inject = ['$scope'];

  /**
  * @namespace PostsController
  */
  function PostsController($scope) {
    var vm = this;

    vm.columns = [];

    activate();


    /**
    * @name activate
    * @desc Actions to be performed when this controller is instantiated
    * @memberOf thinkster.posts.controllers.PostsController
    */
    function activate() {
      $scope.$watchCollection(function () { return $scope.posts; }, render);
      $scope.$watch(function () { return $(window).width(); }, render);
    }


    /**
    * @name calculateNumberOfColumns
    * @desc Calculate number of columns based on screen width
    * @returns {Number} The number of columns containing Posts
    * @memberOf thinkster.posts.controllers.PostsControllers
    */
    function calculateNumberOfColumns() {
      var width = $(window).width();

      if (width >= 1200) {
        return 4;
      } else if (width >= 992) {
        return 3;
      } else if (width >= 768) {
        return 2;
      } else {
        return 1;
      }
    }


    /**
    * @name approximateShortestColumn
    * @desc An algorithm for approximating which column is shortest
    * @returns The index of the shortest column
    * @memberOf thinkster.posts.controllers.PostsController
    */
    function approximateShortestColumn() {
      var scores = vm.columns.map(columnMapFn);

      return scores.indexOf(Math.min.apply(this, scores));


      /**
      * @name columnMapFn
      * @desc A map function for scoring column heights
      * @returns The approximately normalized height of a given column
      */
      function columnMapFn(column) {
        var lengths = column.map(function (element) {
          return element.content.length;
        });

        return lengths.reduce(sum, 0) * column.length;
      }


      /**
      * @name sum
      * @desc Sums two numbers
      * @params {Number} m The first number to be summed
      * @params {Number} n The second number to be summed
      * @returns The sum of two numbers
      */
      function sum(m, n) {
        return m + n;
      }
    }


    /**
    * @name render
    * @desc Renders Posts into columns of approximately equal height
    * @param {Array} current The current value of `vm.posts`
    * @param {Array} original The value of `vm.posts` before it was updated
    * @memberOf thinkster.posts.controllers.PostsController
    */
    function render(current, original) {
      if (current !== original) {
        vm.columns = [];

        for (var i = 0; i < calculateNumberOfColumns(); ++i) {
          vm.columns.push([]);
        }

        for (var i = 0; i < current.length; ++i) {
          var column = approximateShortestColumn();

          vm.columns[column].push(current[i]);
        }
      }
    }
  }
})();
```

Добавьте этот файл в `javascripts.html`:

```html
<script type="text/javascript" src="{% static 'javascripts/posts/controllers/posts.controller.js' %}"></script>
```

Не будем тратить время изучая этот контроллер построчно. Достаточно отметить, что этот контроллер представляет собой алгоритм, гарантирующий, что все столбцы с постами будут иметь примерно равную высоту.

Стоит обратить внимание только на одну строку:

```javascript
$scope.$watchCollection(function () { return $scope.posts; }, render);
```

Поскольку у нас нет прямого доступа к ViewModel, в которой хранится `posts`, мы отслеживаем `$scope.posts`, а не `vm.posts`. Кроме того, здесь мы используем `$watchCollection`, поскольку `$scope.posts` является массивом. `$watch` следит за ссылкой на объект, а не за реальным значением. `$watchCollection` отслеживает за изменениями значений массива. Если бы мы использовали `$watch` вместо `$watchCollection`, изменения, вызванные `$scope.posts.shift()` и `$scope.posts.unshift()` не приводили бы к срабытыванию наблюдателя.

## Создаём шаблон для директивы постов

В нашей директиве мы определили в `templateUrl` адрес, который не соответствует ни одному из наших существующих шаблонов. Давайте создадим его.

Создайте файл `static/templates/posts/posts.html` со следующим содержимым:

```html
<div class="row" ng-cloak>
  <div ng-repeat="column in vm.columns">
    <div class="col-xs-12 col-sm-6 col-md-4 col-lg-3">
      <div ng-repeat="post in column">
        <post post="post"></post>
      </div>
    </div>
  </div>

  <div ng-hide="vm.columns && vm.columns.length">
    <div class="col-sm-12 no-posts-here">
      <em>The are no posts here.</em>
    </div>
  </div>
</div>
```

Стоит отметить следующее:

1. Мы использовали директиву `ng-cloak` для предотвращения мерцаний, поскольку эта директива будет использоваться на главной странице.
2. Нам необходимо создать директиву `post` для отображения каждого отдельного поста.
3. Если постов нет, то мы выдаем сообщение, информирующее пользователя об этом.

## Создаём директиву для отображения одного Postа

В шаблоне для директивы `posts` мы используем другую директиву под названием `post`. Давайте создадим её.

Создайте файл `static/javascripts/posts/directives/post.directive.js` со следующим содержимым:

```javascript
/**
* Post
* @namespace thinkster.posts.directives
*/
(function () {
  'use strict';

  angular
    .module('thinkster.posts.directives')
    .directive('post', post);

  /**
  * @namespace Post
  */
  function post() {
    /**
    * @name directive
    * @desc The directive to be returned
    * @memberOf thinkster.posts.directives.Post
    */
    var directive = {
      restrict: 'E',
      scope: {
        post: '='
      },
      templateUrl: '/static/templates/posts/post.html'
    };

    return directive;
  }
})();
```

Добавьте этот файл в `javascripts.html`:

```html
<script type="text/javascript" src="{% static 'javascripts/posts/directives/post.directive.js' %}"></script>
```

В этом фрагменте кода нет ничего нового, что бы стоило обсудить. Эта директива почти идентична предыдущей. Единственное отличие заключается в том, что мы использовали другой шаблон.

## Создаём шаблон для директивы одного поста

Как и для директивы `posts`, нам нужно создать шаблон для директивы `post`.

Создайте `static/templates/posts/post.html` со следующим содержимым:

```html
<div class="row">
  <div class="col-sm-12">
    <div class="well">
      <div class="post">
        <div class="post__meta">
          <a href="/+{{ post.author.username }}">
            +{{ post.author.username }}
          </a>
        </div>

        <div class="post__content">
          {{ post.content }}
        </div>
      </div>
    </div>
  </div>
</div>
```

## Добавляем немного CSS

Мы хотим добавить несколько простых стилей, чтобы наши посты выглядели лучше. Откройте `static/stylesheets/styles.css` и добавьте следующий код:

```css
.no-posts-here {
  text-align: center;
}

.post {}

.post .post__meta {
  font-weight: bold;
  text-align: right;
  padding-bottom: 19px;
}

.post .post__meta a:hover {
  text-decoration: none;
}
```

## Контрольная точка

Проверить, что Вы все сделали правильно, можно перейдя по адресу `http://localhost:8000/` в своём браузере. Вы должны увидеть `Post` объект, созданный Вами в конце предыдущего раздела!

Это также подтвердит правильность работы `PostViewSet` из предыдущего раздела.
