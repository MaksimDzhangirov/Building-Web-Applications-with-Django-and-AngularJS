# Реализация возможности выхода пользователей из системы

Учитывая то, что пользователи могут зарегистрироваться и войти в систему, логично предположить, что они хотели бы также иметь способ выйти из системы. Пользователи могут начать возмущаться, если не смогут выйти из системы.

## Создаём представление для API выхода из системы

Давайте реализуем последнее представление, связанное с API для  аутентификации.

Откройте файл `authentication/views.py` и добавьте следующие импорты и класс:

```python
from django.contrib.auth import logout

from rest_framework import permissions

class LogoutView(views.APIView):
    permission_classes = (permissions.IsAuthenticated,)

    def post(self, request, format=None):
        logout(request)

        return Response({}, status=status.HTTP_204_NO_CONTENT)
```

В этот раз стоит упомянуть только о некоторых моментах.

```python
permission_classes = (permissions.IsAuthenticated,)
```

Только аутентифицированным пользователям можно обращаться к этой конечной точке API. Эту логику работы позволяет реализовать `permissions.IsAuthenticated` Django REST фреймворка. Если пользователь не аутентифицирован, то он получит 403 ошибку.

```python
logout(request)
```

Если пользователь аутентифицирован, то всё что нам нужно сделать - это вызвать Django метод `logout()`.

```python
return Response({}, status=status.HTTP_204_NO_CONTENT)
```

Нет смысла возвращать что-то при выходе пользователя из системы, поэтому мы просто возвращаем пустой ответ от сервера с кодом статуса 200.

Переходим к URL-шаблонам.

Опять откройте файл `thinkster_django_angular_boilerplate/urls.py` и добавьте следующий импорт и URL:

```python
from authentication.views import LogoutView

urlpatterns = patterns(
    # ...
    url(r'^api/v1/auth/logout/$', LogoutView.as_view(), name='logout'),
    #...
)
```

## Выход из системы: AngularJS служба

Последний метод, который Вам нужно добавить в Вашу службу `Authentication` - это метод `logout()`.

Добавьте следующий метод в службу `Authentication` в файл `authentication.service.js`:

```javascript
/**
 * @name logout
 * @desc Try to log the user out
 * @returns {Promise}
 * @memberOf thinkster.authentication.services.Authentication
 */
function logout() {
  return $http.post('/api/v1/auth/logout/')
    .then(logoutSuccessFn, logoutErrorFn);

  /**
   * @name logoutSuccessFn
   * @desc Unauthenticate and redirect to index with page reload
   */
  function logoutSuccessFn(data, status, headers, config) {
    Authentication.unauthenticate();

    window.location = '/';
  }

  /**
   * @name logoutErrorFn
   * @desc Log "Epic failure!" to the console
   */
  function logoutErrorFn(data, status, headers, config) {
    console.error('Epic failure!');
  }
}
```
Как обычно не забудьте добавить метод `logout()` в службу `Authentication`:

```javascript
var Authentication = {
  getAuthenticatedAccount: getAuthenticatedAccount,
  isAuthenticated: isAuthenticated,
  login: login,
  logout: logout,
  register: register,
  setAuthenticatedUser: setAuthenticatedUser,
  unauthenticate: unauthenticate
};
```

## Управляем навигационной панелью с помощьью NavbarController

Нам не нужен `LogoutController` или `logout.html`. Вместо этого навигационная панель уже содержит ссылку для выхода из системы в случае, когда пользователь аутентифицирован. Мы создадим `NavbarController` для обработки события `onclick` по кнопке выхода из системы и добавим в саму ссылку атрибут `ng-click`.

Создайте файл в каталоге `static/javascripts/layout/controllers/` под названием `navbar.controller.js` и добавьте в него следующий код:

```javascript
/**
* NavbarController
* @namespace thinkster.layout.controllers
*/
(function () {
  'use strict';

  angular
    .module('thinkster.layout.controllers')
    .controller('NavbarController', NavbarController);

  NavbarController.$inject = ['$scope', 'Authentication'];

  /**
  * @namespace NavbarController
  */
  function NavbarController($scope, Authentication) {
    var vm = this;

    vm.logout = logout;

    /**
    * @name logout
    * @desc Log the user out
    * @memberOf thinkster.layout.controllers.NavbarController
    */
    function logout() {
      Authentication.logout();
    }
  }
})();
```

Откройте `templates/navbar.html` и добавьте директиву `ng-controller` со значением `NavbarController as vm` в тег `<nav />` как показано ниже:

```html
<nav class="navbar navbar-default" role="navigation" ng-controller="NavbarController as vm">
```
В этом же файле (`templates/navbar.html`), найдите ссылку для выхода из системы и добавьте в неё `ng-click="vm.logout()"` следующим образом:

```html
<li><a href="javascript:void(0)" ng-click="vm.logout()">Logout</a></li>
```

## Модули, отвечающие за расположение информации на сайте

В этот раз нам нужно добавить несколько новых модулей.

Создайте файл в каталоге `static/javascripts/layout/` с названием `layout.module.js` и добавьте в него следующее содержимое:

```javascript
(function () {
  'use strict';

  angular
    .module('thinkster.layout', [
      'thinkster.layout.controllers'
    ]);

  angular
    .module('thinkster.layout.controllers', []);
})();
```

Также не забудьте обновить `static/javascripts/thinkster.js`:

```javascript
angular
  .module('thinkster', [
    'thinkster.config',
    'thinkster.routes',
    'thinkster.authentication',
    'thinkster.layout'
  ]);
```

## Добавляем новые .js файлы

В этот раз нужно добавить несколько новых JavaScript файлов. Откройте `javascripts.html` и добавьте следующее:

```html
<script type="text/javascript" src="{% static 'javascripts/layout/layout.module.js' %}"></script>
<script type="text/javascript" src="{% static 'javascripts/layout/controllers/navbar.controller.js' %}"></script>
```

## Контрольная точка

Если Вы перейдёте по адресу `http://localhost:8000/` в Вашем браузере, то увидите, что до сих пор авторизованы. В противном случае, Вам нужно будет снова войди в систему.

Вы можете проверить работоспособность функционала выхода из системы, нажав на кнопку выхода из системы в навигационной панели. При этом  страница и навигационная панель обновится к виду, который используется для не вошедших в систему пользователей.
