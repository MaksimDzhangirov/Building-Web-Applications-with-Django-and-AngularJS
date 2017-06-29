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

Определив модуль, мы готовы создать Profile службу, которая будет взаимодействовать с нашим API.

Создайте файл `static/javascripts/profiles/services/profile.service.js` со следующим содержимым:

```javascript
/**
* Profile
* @namespace thinkster.profiles.services
*/
(function () {
  'use strict';

  angular
    .module('thinkster.profiles.services')
    .factory('Profile', Profile);

  Profile.$inject = ['$http'];

  /**
  * @namespace Profile
  */
  function Profile($http) {
    /**
    * @name Profile
    * @desc The factory to be returned
    * @memberOf thinkster.profiles.services.Profile
    */
    var Profile = {
      destroy: destroy,
      get: get,
      update: update
    };

    return Profile;

    /////////////////////

    /**
    * @name destroy
    * @desc Destroys the given profile
    * @param {Object} profile The profile to be destroyed
    * @returns {Promise}
    * @memberOf thinkster.profiles.services.Profile
    */
    function destroy(profile) {
      return $http.delete('/api/v1/accounts/' + profile.id + '/');
    }


    /**
    * @name get
    * @desc Gets the profile for user with username `username`
    * @param {string} username The username of the user to fetch
    * @returns {Promise}
    * @memberOf thinkster.profiles.services.Profile
    */
    function get(username) {
      return $http.get('/api/v1/accounts/' + username + '/');
    }


    /**
    * @name update
    * @desc Update the given profile
    * @param {Object} profile The profile to be updated
    * @returns {Promise}
    * @memberOf thinkster.profiles.services.Profile
    */
    function update(profile) {
      return $http.put('/api/v1/accounts/' + profile.username + '/', profile);
    }
  }
})();
```

В этом фрагменте кода нет ничего особенного. Каждый из этих вызовов API - это базовая CRUD операция, поэтому для их реализаций не потребуется писать много кода.

Добавьте этот файл в `javascripts.html`:

```html
<script type="text/javascript" src="{% static 'javascripts/profiles/services/profile.service.js' %}"></script>
```

## Создаём интерфейс для профилей пользователей

Создайте файл `static/templates/profiles/profile.html` со следующим содержимым:

```html
<div class="profile" ng-show="vm.profile">
  <div class="jumbotron profile__header">
    <h1 class="profile__username">+{{ vm.profile.username }}</h1>
    <p class="profile__tagline">{{ vm.profile.tagline }}</p>
  </div>

  <posts posts="vm.posts"></posts>
</div>
```

Этот фрагмент кода отображает заголовок с именем пользователя и девиз владельца пользователя, после чего следует список его постов. Посты отображаются с помощью директивы, созданной нами ранее, для главной страницы.

## Управляем интерфейсом профиля с помощью ProfileController

Следующим шагом будет создание контроллера. Он будет использовать только что созданную службу вместе со службой `Post` для получения данных, которые мы хотим отобразить.

Создайте файл `static/javascripts/profiles/controllers/profile.controller.js` со следующим содержимым:

```javascript
/**
* ProfileController
* @namespace thinkster.profiles.controllers
*/
(function () {
  'use strict';

  angular
    .module('thinkster.profiles.controllers')
    .controller('ProfileController', ProfileController);

  ProfileController.$inject = ['$location', '$routeParams', 'Posts', 'Profile', 'Snackbar'];

  /**
  * @namespace ProfileController
  */
  function ProfileController($location, $routeParams, Posts, Profile, Snackbar) {
    var vm = this;

    vm.profile = undefined;
    vm.posts = [];

    activate();

    /**
    * @name activate
    * @desc Actions to be performed when this controller is instantiated
    * @memberOf thinkster.profiles.controllers.ProfileController
    */
    function activate() {
      var username = $routeParams.username.substr(1);

      Profile.get(username).then(profileSuccessFn, profileErrorFn);
      Posts.get(username).then(postsSuccessFn, postsErrorFn);

      /**
      * @name profileSuccessProfile
      * @desc Update `profile` on viewmodel
      */
      function profileSuccessFn(data, status, headers, config) {
        vm.profile = data.data;
      }


      /**
      * @name profileErrorFn
      * @desc Redirect to index and show error Snackbar
      */
      function profileErrorFn(data, status, headers, config) {
        $location.url('/');
        Snackbar.error('That user does not exist.');
      }


      /**
        * @name postsSucessFn
        * @desc Update `posts` on viewmodel
        */
      function postsSuccessFn(data, status, headers, config) {
        vm.posts = data.data;
      }


      /**
        * @name postsErrorFn
        * @desc Show error snackbar
        */
      function postsErrorFn(data, status, headers, config) {
        Snackbar.error(data.data.error);
      }
    }
  }
})();
```

Добавьте этот файл в `javascripts.html`:

```html
<script type="text/javascript" src="{% static 'javascripts/profiles/controllers/profile.controller.js' %}"></script>
```

## Создаём маршрут для просмотра профилей пользователей

Откройте `static/javascripts/thinkster.routes.js` и добавьте следующий маршрут:

```javascript
.when('/+:username', {
  controller: 'ProfileController',
  controllerAs: 'vm',
  templateUrl: '/static/templates/profiles/profile.html'
})
```

## Контрольная точка

Чтобы просмотреть свой профиль, перейдите в Вашем браузере по адресу `http://localhost:8000/+<username>`. Если страница отображается, то Вы всё сделали правильно!

