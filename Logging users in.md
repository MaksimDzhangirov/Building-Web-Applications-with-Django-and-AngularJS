# Реализация возможности входа пользователей в систему

Теперь когда пользователи могут зарегистрироваться, им нужен способ входа в систему. Так получилось, что эта часть не реализована в нашей системе регистрации. Как только пользователь зарегистрировался, мы должны автоматически позволить ему войти в систему.

Для начала создадим представления позволяющие осуществлять вход и выход в/из системы. После этого процесс разработки будет аналогичен тому, который использовался для системы регистрации: службы, контроллеры и т. д.

## Создаём представление для API входа в систему

Откройте `authentication/views.py` и добавьте в него следующий код:

```python
import json

from django.contrib.auth import authenticate, login

from rest_framework import status, views
from rest_framework.response import Response

class LoginView(views.APIView):
    def post(self, request, format=None):
        data = json.loads(request.body)

        email = data.get('email', None)
        password = data.get('password', None)

        account = authenticate(email=email, password=password)

        if account is not None:
            if account.is_active:
                login(request, account)

                serialized = AccountSerializer(account)

                return Response(serialized.data)
            else:
                return Response({
                    'status': 'Unauthorized',
                    'message': 'This account has been disabled.'
                }, status=status.HTTP_401_UNAUTHORIZED)
        else:
            return Response({
                'status': 'Unauthorized',
                'message': 'Username/password combination invalid.'
            }, status=status.HTTP_401_UNAUTHORIZED)
```

Это самый большой фрагмент кода по сравнению с предыдущими, но мы исследовать его таким же образом как и предыдущие: рассматривая только новые, не встречавшиеся ранее фрагменты и игнориря те, с которыми уже сталкивались.

```python
class LoginView(views.APIView):
```

Вы наверное заметили, что в этот раз мы не используем обобщенное представление. Посольку это представление не реализует обобщенное действие, такое как создание или обновление объекта, нам необходимо начать с чего-то более простого. Мы будем использовать `views.APIView` Django REST фреймворка. Хотя `APIView` не реализует всю логику работы приложения за нас, его использовать выгоднее, чем стандартные Django представления. В частности, `views.APIView` созданы специально для обработки AJAX запросов. Это позволяет съэкономить нам уйму времени.

```python
def post(self, request, format=None):
```

В отличие от обобщенных представлений, мы должны обрабатывать каждый HTTP метод сами. Обычно вход в систему реализуется с помощью POST запроса, поэтому мы должны переопределить метод `self.post()`:

```python
account = authenticate(email=email, password=password)
```

Django предоставляет хороший набор утилит для аутентификации пользователей. Метод `authenticate()` - это первая утилита, которую мы рассмотрим. `authenticate()` принимает в качестве параметров электронную почту и пароль. Затем Django проверяет существует ли в базе данных учетная запись `Account` с указанной электронной почтой `email`. Если указанный адрес электронной почты найден в базе, Django пытается проверить введенный пароль. Если введены правильный адрес электронной почты и пароль, то метод `authenticate()` возвращает найденный объект `Account`. Если на любом из этих этапов происходит ошибка, метод `authenticate()` возвратит `None`.

```python
if account is not None:
    # ...
else:
    return Response({
        'status': 'Unauthorized',
        'message': 'Username/password combination invalid.'
    }, status=status.HTTP_401_UNAUTHORIZED)
```

В случае, когда `authenticate()` возвращает `None`, мы возвращаем код состояния HTTP 401 и сообщаем пользователю, что указанная ими комбинация электронной почты/пароля неверна.

```python
if account.is_active:
    # ...
else:
    return Response({
        'status': 'Unauthorized',
        'message': 'This account has been disabled.'
    }, status=status.HTTP_401_UNAUTHORIZED)
```

Если учетная запись пользователя по какой-то причине не активна, мы возвращаем код состояния HTTP 401. В этом случае мы просто сообщаем, что учетная ззапись была деактивирована.

```python
login(request, account)
```

Если метод `authenticate()` вернул учетную запись и она активна, то мы используем утилиту Django, чтобы создать новую сессию для этого пользователя.

```python
serialized = AccountSerializer(account)

return Response(serialized.data)
```

Мы хотим сохранить некоторую информацию об этом пользователе в браузере при успешном входе в систему, поэтому мы сериализуем объект `Account`, найденный методом `authenticate()` и возвращаем полученный JSON в качестве ответа от сервера.

## Добавляем конечную точку для API входа в систему

Как и в случае с `AccountViewSet`, нам надо добавить URL для `LoginView`.

Откройте `thinkster_django_angular_boilerplate/urls.py` и добавьте следующий URL-шаблон между `^/api/v1/` и `^`:

```python
from authentication.views import LoginView

urlpatterns = patterns(
    # ...
    url(r'^api/v1/auth/login/$', LoginView.as_view(), name='login'),
    # ...
)
```

## Служба аутентификации

Давайте добавим ещё несколько методов в нашу службу аутентификации. Мы сделаем это в два этапа. Сначала мы добавим метод `login()`, а затем некоторые вспомогательные методы для хранения данных сессии в браузере.

Откройте `static/javascripts/authentication/services/authentication.service.js` и добавьте следующий метод в объект `Authentication`, созданный нами ранее:

```javascript
/**
 * @name login
 * @desc Try to log in with email `email` and password `password`
 * @param {string} email The email entered by the user
 * @param {string} password The password entered by the user
 * @returns {Promise}
 * @memberOf thinkster.authentication.services.Authentication
 */
function login(email, password) {
  return $http.post('/api/v1/auth/login/', {
    email: email, password: password
  });
}
```
Не забудьте добавить его в службу:

```javascript
var Authentication = {
  login: login,
  register: register
};
```

Подобно методу `register()`, `login()` осуществляет AJAX запрос к нашему API и возвращает promise.

Теперь давайте поговорим о некоторых вспомогательных методах, которые нам нужны для работы с сессионной информацией на стороне клиента.

Мы хотим отображать информацию о пользователе, аутентифицированном в настоящий момент, в навигационной панели вверху страницы. Это означает, что нам нужен способ сохранить ответ от сервера, возвращаемый методом `login()`. Также необходим способ извлечения данных аутентифицированного пользователя. Нам необходим способ, позволяющий сделать пользователя не аутентифицированным в браузере. Наконец, хорошо бы было иметь возможность легко проверять аутентифицирован ли текущий пользователь.

*Замечание: Сделать пользователя не аутентифицированным, это не то же самое, что осуществить выход пользователя из системы. Когда пользователь выходит из системы, нам нужен способ удалить все оставшиеся данные сессии из клиентского приложения.*

С учётом этих требований, я предлагаю использовать четыре метода: `getAuthenticatedAccount`, `isAuthenticated`, `setAuthenticatedAccount`, и `unauthenticate`.

Давайте сейчас их реализуем. Добавьте следующие функции в службу `Authentication`:

```javascript
/**
 * @name getAuthenticatedAccount
 * @desc Return the currently authenticated account
 * @returns {object|undefined} Account if authenticated, else `undefined`
 * @memberOf thinkster.authentication.services.Authentication
 */
function getAuthenticatedAccount() {
  if (!$cookies.authenticatedAccount) {
    return;
  }

  return JSON.parse($cookies.authenticatedAccount);
}
```

Если не существует куки `authenticatedAccount` (который устанавливается в функции `setAuthenticatedAccount()`), то ничего не возвращаем. В противном случае возвращается спарсенный из куки объект пользователя.

```javascript
/**
 * @name isAuthenticated
 * @desc Check if the current user is authenticated
 * @returns {boolean} True is user is authenticated, else false.
 * @memberOf thinkster.authentication.services.Authentication
 */
function isAuthenticated() {
  return !!$cookies.authenticatedAccount;
}
```

Возвращаем логическое значение куки `authenticatedAccount`.

```javascript
/**
 * @name setAuthenticatedAccount
 * @desc Stringify the account object and store it in a cookie
 * @param {Object} user The account object to be stored
 * @returns {undefined}
 * @memberOf thinkster.authentication.services.Authentication
 */
function setAuthenticatedAccount(account) {
  $cookies.authenticatedAccount = JSON.stringify(account);
}
```

Присваиваем куки `authenticatedAccount` значение объекта `account`, преобразованного в строку.

```javascript
/**
 * @name unauthenticate
 * @desc Delete the cookie where the user object is stored
 * @returns {undefined}
 * @memberOf thinkster.authentication.services.Authentication
 */
function unauthenticate() {
  delete $cookies.authenticatedAccount;
}
```

Удаляем куки `authenticatedAccount`.

Опять не забудьте добавить эти методы в службу:

```javascript
var Authentication = {
  getAuthenticatedAccount: getAuthenticatedAccount,
  isAuthenticated: isAuthenticated,
  login: login,
  register: register,
  setAuthenticatedAccount: setAuthenticatedAccount,
  unauthenticate: unauthenticate
};
```

Прежде чем переходить к разработке интерфейса для входа в систему, давайте быстро обновим метод `login` службы `Authentication`, чтобы использовать один из этих вспомогательных методов. Замените `Authentication.login` следующим кодом:

```javascript
/**
 * @name login
 * @desc Try to log in with email `email` and password `password`
 * @param {string} email The email entered by the user
 * @param {string} password The password entered by the user
 * @returns {Promise}
 * @memberOf thinkster.authentication.services.Authentication
 */
function login(email, password) {
  return $http.post('/api/v1/auth/login/', {
    email: email, password: password
  }).then(loginSuccessFn, loginErrorFn);

  /**
   * @name loginSuccessFn
   * @desc Set the authenticated account and redirect to index
   */
  function loginSuccessFn(data, status, headers, config) {
    Authentication.setAuthenticatedAccount(data.data);

    window.location = '/';
  }

  /**
   * @name loginErrorFn
   * @desc Log "Epic failure!" to the console
   */
  function loginErrorFn(data, status, headers, config) {
    console.error('Epic failure!');
  }
}
```

## Создаём интерфейс для входа в систему

Теперь когда создана функция `Authentication.login()` для входа пользователя в систему, давайте создадим форму для входа в систему. Откройте `static/templates/authentication/login.html` и добавьте следующий HTML код:

```html
<div class="row">
  <div class="col-md-4 col-md-offset-4">
    <h1>Login</h1>

    <div class="well">
      <form role="form" ng-submit="vm.login()">
        <div class="alert alert-danger" ng-show="error" ng-bind="error"></div>

        <div class="form-group">
          <label for="login__email">Email</label>
          <input type="text" class="form-control" id="login__email" ng-model="vm.email" placeholder="ex. john@example.com" />
        </div>

        <div class="form-group">
          <label for="login__password">Password</label>
          <input type="password" class="form-control" id="login__password" ng-model="vm.password" placeholder="ex. thisisnotgoogleplus" />
        </div>

        <div class="form-group">
          <button type="submit" class="btn btn-primary">Submit</button>
        </div>
      </form>
    </div>
  </div>
</div>
```

