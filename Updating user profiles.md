# Обновляем профили пользователей

Последний функционал, который будет реализован в этом учебном пособии - это возможность для пользователя обновлять свой профиль. Мы реализуем обновления для небольшого количества полей - имени и фамилии пользователя, адреса электронной почты и девиза, но Вы, поняв суть, сможете добавить в этот список другие поля, если захотите.

## Контроллер ProfileSettings

Для начала создайте файл `static/javascripts/profiles/controllers/profile-settings.controller.js` и добавьте следующее содержимое:

```javascript
/**
* ProfileSettingsController
* @namespace thinkster.profiles.controllers
*/
(function () {
  'use strict';

  angular
    .module('thinkster.profiles.controllers')
    .controller('ProfileSettingsController', ProfileSettingsController);

  ProfileSettingsController.$inject = [
    '$location', '$routeParams', 'Authentication', 'Profile', 'Snackbar'
  ];

  /**
  * @namespace ProfileSettingsController
  */
  function ProfileSettingsController($location, $routeParams, Authentication, Profile, Snackbar) {
    var vm = this;

    vm.destroy = destroy;
    vm.update = update;

    activate();


    /**
    * @name activate
    * @desc Actions to be performed when this controller is instantiated.
    * @memberOf thinkster.profiles.controllers.ProfileSettingsController
    */
    function activate() {
      var authenticatedAccount = Authentication.getAuthenticatedAccount();
      var username = $routeParams.username.substr(1);

      // Redirect if not logged in
      if (!authenticatedAccount) {
        $location.url('/');
        Snackbar.error('You are not authorized to view this page.');
      } else {
        // Redirect if logged in, but not the owner of this profile.
        if (authenticatedAccount.username !== username) {
          $location.url('/');
          Snackbar.error('You are not authorized to view this page.');
        }
      }

      Profile.get(username).then(profileSuccessFn, profileErrorFn);

      /**
      * @name profileSuccessFn
      * @desc Update `profile` for view
      */
      function profileSuccessFn(data, status, headers, config) {
        vm.profile = data.data;
      }

      /**
      * @name profileErrorFn
      * @desc Redirect to index
      */
      function profileErrorFn(data, status, headers, config) {
        $location.url('/');
        Snackbar.error('That user does not exist.');
      }
    }


    /**
    * @name destroy
    * @desc Destroy this user's profile
    * @memberOf thinkster.profiles.controllers.ProfileSettingsController
    */
    function destroy() {
      Profile.destroy(vm.profile.username).then(profileSuccessFn, profileErrorFn);

      /**
      * @name profileSuccessFn
      * @desc Redirect to index and display success snackbar
      */
      function profileSuccessFn(data, status, headers, config) {
        Authentication.unauthenticate();
        window.location = '/';

        Snackbar.show('Your account has been deleted.');
      }


      /**
      * @name profileErrorFn
      * @desc Display error snackbar
      */
      function profileErrorFn(data, status, headers, config) {
        Snackbar.error(data.error);
      }
    }


    /**
    * @name update
    * @desc Update this user's profile
    * @memberOf thinkster.profiles.controllers.ProfileSettingsController
    */
    function update() {
      Profile.update(vm.profile).then(profileSuccessFn, profileErrorFn);

      /**
      * @name profileSuccessFn
      * @desc Show success snackbar
      */
      function profileSuccessFn(data, status, headers, config) {
        Snackbar.show('Your profile has been updated.');
      }


      /**
      * @name profileErrorFn
      * @desc Show error snackbar
      */
      function profileErrorFn(data, status, headers, config) {
        Snackbar.error(data.error);
      }
    }
  }
})();
```

Не забудьте добавить этот файл в `javascripts.html`:

```html
<script type="text/javascript" src="{% static 'javascripts/profiles/controllers/profile-settings.controller.js' %}"></script>
```

Здесь мы создали два метода, которые будут доступны в шаблоне: `update` и `destroy`. Как понятно из названий методов `update` позволит пользователю обновить свой профиль, а `destroy` удалит учетную запись пользователя.

Большая часть кода этого контроллера должна быть Вам знакома, но для улучшения усвоения материала давайте пройдемся по методам, которые, которые мы создали.

```javascript
/**
 * @name activate
 * @desc Actions to be performed when this controller is instantiated.
 * @memberOf thinkster.profiles.controllers.ProfileSettingsController
 */
function activate() {
  var authenticatedAccount = Authentication.getAuthenticatedAccount();
  var username = $routeParams.username.substr(1);

  // Redirect if not logged in
  if (!authenticatedAccount) {
    $location.url('/');
    Snackbar.error('You are not authorized to view this page.');
  } else {
    // Redirect if logged in, but not the owner of this profile.
    if (authenticatedAccount.username !== username) {
      $location.url('/');
      Snackbar.error('You are not authorized to view this page.');
    }
  }

  Profile.get(username).then(profileSuccessFn, profileErrorFn);

  /**
   * @name profileSuccessFn
   * @desc Update `profile` for view
   */
  function profileSuccessFn(data, status, headers, config) {
    vm.profile = data.data;
  }

  /**
   * @name profileErrorFn
   * @desc Redirec to index
   */
  function profileErrorFn(data, status, headers, config) {
    $location.url('/');
    Snackbar.error('That user does not exist.');
  }
}
```

В `activate` мы используем знакомый нам принцип. Поскольку на этой странице разрешено выполнять опасные операции, мы должны убедиться, что текущий пользователь имеет право просматривать эту страницу. Для этого мы проверяем аутентифицирован ли пользователь, а затем является ли аутентифицированный пользователь владельцем профиля. Если любое из этих условий ложно, то мы перенаправляем пользователя на главную страницу, выдавая ошибку на информационной панели в нижней части экрана, которая сообщает о том, что пользователю не разрешено просматривать эту страницу.

Если процесс авторизации завершился успешно, мы просто извлекаем профиль пользователя с сервера и позволяем пользователю делать с ним всё, что он пожелает.

```javascript
/**
 * @name destroy
 * @desc Destroy this user's profile
 * @memberOf thinkster.profiles.controllers.ProfileSettingsController
 */
function destroy() {
  Profile.destroy(vm.profile).then(profileSuccessFn, profileErrorFn);

  /**
   * @name profileSuccessFn
   * @desc Redirect to index and display success snackbar
   */
  function profileSuccessFn(data, status, headers, config) {
    Authentication.unauthenticate();
    window.location = '/';

    Snackbar.show('Your account has been deleted.');
  }


  /**
   * @name profileErrorFn
   * @desc Display error snackbar
   */
  function profileErrorFn(data, status, headers, config) {
    Snackbar.error(data.error);
  }
}
```

Если пользователь удалил свой профиль, то мы должны сделать его не аутентифицированным и перенаправить на главную страницу, обновив её содержимое. Это приведет к тому, что навигационная панель будет отображаться в виде, в котором она доступна для не вышедших в систему пользователей.

Если по какой-то причине при удалении профиля сервер вернул код статуса ошибки, мы просто отображаем сообщение об ошибке, возвращённое сервером, на информационной панели в нижней части экрана. Мы не выполняем никаких других действий, поскольку единственная реальная причина из-за которой этот запрос может завершиться с ошибкой заключается в том, что пользователь не имеет права удалять этот профиль, но мы учли этот случай в методе `activate`.

```javascript
/**
 * @name update
 * @desc Update this user's profile
 * @memberOf thinkster.profiles.controllers.ProfileSettingsController
 */
function update() {
  Profile.update(vm.profile).then(profileSuccessFn, profileErrorFn);

  /**
   * @name profileSuccessFn
   * @desc Show success snackbar
   */
  function profileSuccessFn(data, status, headers, config) {
    Snackbar.show('Your profile has been updated.');
  }


  /**
   * @name profileErrorFn
   * @desc Show error snackbar
   */
  function profileErrorFn(data, status, headers, config) {
    Snackbar.error(data.error);
  }
}
```

Метод `update()` очень простой. Не зависимо от того завершился запрос успешно или с ошибкой, мы отображаем информационную панель в нижней части экрана с соответствующим сообщением.

## Шаблон для страницы профиля

Как обычно, после того как мы создали контроллер, нам нужно реализовать соответствующий шаблон.

Создайте файл `static/templates/profiles/settings.html` со следующим содержимым:

```html
<div class="col-md-4 col-md-offset-4">
  <div class="well" ng-show="vm.profile">
    <form role="form" class="settings" ng-submit="vm.update()">
      <div class="form-group">
        <label for="settings__email">Email</label>
        <input type="text" class="form-control" id="settings__email" ng-model="vm.profile.email" placeholder="ex. john@example.com" />
      </div>

      <div class="form-group">
        <label for="settings__password">New Password</label>
        <input type="password" class="form-control" id="settings__password" ng-model="vm.profile.password" placeholder="ex. notgoogleplus" />
      </div>

      <div class="form-group">
        <label for="settings__confirm-password">Confirm Password</label>
        <input type="password" class="form-control" id="settings__confirm-password" ng-model="vm.profile.confirm_password" placeholder="ex. notgoogleplus" />
      </div>

      <div class="form-group">
        <label for="settings__username">Username</label>
        <input type="text" class="form-control" id="settings__username" ng-model="vm.profile.username" placeholder="ex. notgoogleplus" />
      </div>

      <div class="form-group">
        <label for="settings__tagline">Tagline</label>
        <textarea class="form-control" id="settings__tagline" ng-model="vm.profile.tagline" placeholder="ex. This is Not Google Plus." />
      </div>

      <div class="form-group">
        <button type="submit" class="btn btn-primary">Submit</button>
        <button type="button" class="btn btn-danger pull-right" ng-click="vm.destroy()">Delete Account</button>
      </div>
    </form>
  </div>
</div>
```

Этот шаблон похож на формы, которые мы создали для страниц регистрации и входа в систему. Здесь нечего обсуждать.

## Маршрут для обновления профиля пользователя

Откройте `static/javascripts/thinkster.routes.js` и добавьте следующий маршрут:

```javascript
// ...
.when('/+:username/settings', {
  controller: 'ProfileSettingsController',
  controllerAs: 'vm',
  templateUrl: '/static/templates/profiles/settings.html'
})
// ...
```

## Контрольная точка

Это наш последний функционал! Теперь Вы можете загрузить страницу с настройками профиля по адресу `http://localhost:8000/+:username/settings` и обновить свой профиль по Вашему усмотрению.

Попробуйте обновить свой девиз. Если всё сработает правильно, то Вы увидите, что Ваш девиз теперь отображается на Вашей странице профиля.
