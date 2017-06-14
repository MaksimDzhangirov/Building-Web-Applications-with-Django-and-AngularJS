# Регистрируем новых пользователей

На данный момент у нас есть модели и сериализаторы, необходимые для работы с данными пользователя. Теперь нам нужно создать систему аутентификации. Это предполагает создание различных представлений и интерфейсов для регистрации, входа в систему и выхода из системы. Мы также коснемся службы `Authentication` для AngularJS и нескольких различных контроллеров.

Поскольку войти в систему могут только существующие пользователи, очевидно, что начать следует с регистрации.

Чтобы зарегистрировать пользователя нам нужна конечная точка API, которая создаст объект `Account`, AngularJS службу для создания AJAX запросов к API и форма для регистрации пользователя. Давайте сначала создадим конечную точку API.

## Создаём Viewset для API учетной записи

Откройте `authentication/views.py` и замените его содержимое следующим кодом:

```python
from rest_framework import permissions, viewsets

from authentication.models import Account
from authentication.permissions import IsAccountOwner
from authentication.serializers import AccountSerializer


class AccountViewSet(viewsets.ModelViewSet):
    lookup_field = 'username'
    queryset = Account.objects.all()
    serializer_class = AccountSerializer

    def get_permissions(self):
        if self.request.method in permissions.SAFE_METHODS:
            return (permissions.AllowAny(),)

        if self.request.method == 'POST':
            return (permissions.AllowAny(),)

        return (permissions.IsAuthenticated(), IsAccountOwner(),)

    def create(self, request):
        serializer = self.serializer_class(data=request.data)

        if serializer.is_valid():
            Account.objects.create_user(**serializer.validated_data)

            return Response(serializer.validated_data, status=status.HTTP_201_CREATED)

        return Response({
            'status': 'Bad request',
            'message': 'Account could not be created with received data.'
        }, status=status.HTTP_400_BAD_REQUEST)
```

Давайте рассмотрим этот фрагмент кода построчно.

```python
class AccountViewSet(viewsets.ModelViewSet):
```

Django REST фреймворк предоставляет функционал под названием `viewset`. Как следует из названия `viewset` является набором представлений. В частности, `ModelViewSet` предоставляет интерфейс для перечисления, создания, извлечения, обновления и удаления объектов заданной модели.

```python
lookup_field = 'username'
queryset = Account.objects.all()
serializer_class = AccountSerializer
```
В этом фрагменте мы определяем `queryset` и сериализатор, с которым будет работать `viewset`. Django REST фреймворк использует указанный queryset и сериализатор для выполнения перечисленных выше действий. Также обратите внимание, что мы определили атрибут `lookup_field`. Как было сказано ранее мы будем использовать атрибут `username` модели `Account` для поиска учетных записей вместо атрибута `id`. Для этого мы переопределяем `lookup_field`.

```python
def get_permissions(self):
    if self.request.method in permissions.SAFE_METHODS:
        return (permissions.AllowAny(),)

    if self.request.method == 'POST':
        return (permissions.AllowAny(),)

    return (permissions.IsAuthenticated(), IsAccountOwner(),)
```

Единственный пользователь, который может вызывать опасные (с точки зрения безопасности - прим. переводчика) методы (такие как `update()` и `delete()`)  - это владелец учетной записи. Сначала мы проверяем аутентифицирован ли пользователь, а затем вызываем пользовательский метод проверки прав пользователя, который мы вскоре напишем. Эта проверка не осуществляется, если HTTP методом является `POST`. Мы хотим, чтобы любой пользователь мог создавать учётную запись.

Если HTTP метод запроса - "безопасный" ("GET", "POST" и т. д.), то кто угодно может использовать эту конечную точку.

```python
def create(self, request):
    serializer = self.serializer_class(data=request.data)

    if serializer.is_valid():
        Account.objects.create_user(**serializer.validated_data)

        return Response(serializer.validated_data, status=status.HTTP_201_CREATED)
    return Response({
        'status': 'Bad request',
        'message': 'Account could not be created with received data.'
    }, status=status.HTTP_400_BAD_REQUEST)
```

Когда Вы создаёте объект, используя метод сериализатора `save()`, присвоение атрибутов объекта происходит без каких-либо преобразований. Это означает, что если пользователь регистрируется, используя пароль `'password'`, то его пароль сохранится как `'password'`. Это плохо по нескольким причинам: 1) Хранение паролей в текстовом виде - это серьезная "дыра" в безопасности системы. 2) Django преобразует пароли, используя хэширование и соли, перед их сравнением, поэтому пользователь не сможет войти в систему, используя строку `'password'` в качестве пароля, если всё оставить как есть.

Мы решаем эту проблему, переопределяя метод `.create()` для этого `viewset` и используя `Account.objects.create_user()` для создания объекта `Account`.

## Создаём пользовательский метод проверки прав IsAccountOwner

Давайте создадим метод проверки прав `IsAccountOwner` для только что созданного нами представления.

Создайте файл под названием `authentication/permissions.py` со следующим содержимым:

```python
from rest_framework import permissions


class IsAccountOwner(permissions.BasePermission):
    def has_object_permission(self, request, view, account):
        if request.user:
            return account == request.user
        return False
```

Это довольно простой метод проверки прав. Если существует пользователь, связанный с текущим запросом, то мы проверяем является ли он тем же объектом, что и `account`. Если не существует пользователя, связанного с текущим запросом, мы просто возвращаем `False`.

## Добавляем конечную точку API

Теперь, когда мы создали представление, нам надо добавить его в URL файл. Откройте `thinkster_django_angular_boilerplate/urls.py` и обновите его, чтобы он выглядел следующим образом:

```python
# .. Импорты
from rest_framework_nested import routers

from authentication.views import AccountViewSet

router = routers.SimpleRouter()
router.register(r'accounts', AccountViewSet)

urlpatterns = patterns(
     '',
    # ... URLы
    url(r'^api/v1/', include(router.urls)),

    url('^.*$', IndexView.as_view(), name='index'),
)
```
> Очень важно, чтобы последний URL в вышеприведенном фрагменте кода был всегда последним. Он называется сквозным или перехватывающим путём. Этот URL-шаблон перехватывает все запросы, не соответствующие ни одному из предыдущих правил и передаёт их в маршрутизатор AngularJS для обработки. Порядок, в котором расположены другие URL, обычно не имеет значения.

## Angular служба для регистрации новых пользователей

Закончив с конечной точкой API, мы можем создать AngularJS службу, которая будет осуществлять взаимодействие между клиентом и сервером.

Создайте файл в каталоге `static/javascripts/authentication/services/` под названием `authentication.service.js` и добавьте в него следующий код:

> Можете не набирать комментарии, когда будете вручную вводить этот код. Чтобы напечатать их уйдет довольно много времени.

```javascript
/**
* Authentication
* @namespace thinkster.authentication.services
*/
(function () {
  'use strict';

  angular
    .module('thinkster.authentication.services')
    .factory('Authentication', Authentication);

  Authentication.$inject = ['$cookies', '$http'];

  /**
  * @namespace Authentication
  * @returns {Factory}
  */
  function Authentication($cookies, $http) {
    /**
    * @name Authentication
    * @desc The Factory to be returned
    */
    var Authentication = {
      register: register
    };

    return Authentication;

    ////////////////////

    /**
    * @name register
    * @desc Try to register a new user
    * @param {string} username The username entered by the user
    * @param {string} password The password entered by the user
    * @param {string} email The email entered by the user
    * @returns {Promise}
    * @memberOf thinkster.authentication.services.Authentication
    */
    function register(email, password, username) {
      return $http.post('/api/v1/accounts/', {
        username: username,
        password: password,
        email: email
      });
    }
  }
})();
```

Рассмотрим каждую строчку:

```javascript
angular
  .module('thinkster.authentication.services')
```

AngularJS позволяет использовать модули. Разбиение приложения на модули - это хорошая практика, поскольку она способствует инкапсуляции и уменьшает зависимости между различными частями приложения. Мы будем постоянно использовать модульную систему Angular во всех частях учебного пособия. Все что Вам нужно знать пока - это то, что эта служба находится внутри модуля `thinkster.authentication.services`.

```javascript
.factory('Authentication', Authentication);
```

В этой строке регистрируется фабрика под названием `Authentication` в модуле из предыдущей строки.

```javascript
function Authentication($cookies, $http) {
```

Здесь мы определяем только что зарегистрированную фабрику. Мы добавляем службы `$cookies` и `$http`, используя внедрение зависимостей. Служба `$cookies` будет использоваться позднее.

```javascript
var Authentication = {
  register: register
};
```

Это личное предпочтение, но мне кажется, что определение Ваших служб в виде именованных объектов, улучшает читаемость кода. Это позволяет расположить детали реализации службы ниже в файле.

```javascript
function register (username, password, email) {
```

На данный момент служба `Authentication` содержит только один метод: `register`, который принимает три аргумента `username`, `password` и `email`. В дальнейшем мы добавим в эту службу другие методы.

```javascript
return $http.post('/api/v1/accounts/', {
  username: username,
  password: password,
  email: email
});
```

Как было сказано ранее, нам необходимо осуществить AJAX запрос к конечной точке API, которую мы создали. В качестве данных мы передаем параметры `username`, `password` и `email`, которые этот метод получает. Мы не хотим обрабатывать ответ от сервера каким-то особым образом, поэтому просто возвращаем ответ, поручая обработку тому, кто вызовет `Authentication.register`.

## Создаём интерфейс для регистрации новых пользователей

Давайте начнем создавать интерфейс, который будут использовать пользователи для регистрации. Создайте файл в каталоге `static/templates/authentication/` под названием `register.html` со следующим кодом:

```html
<div class="row">
  <div class="col-md-4 col-md-offset-4">
    <h1>Register</h1>

    <div class="well">
      <form role="form" ng-submit="vm.register()">
        <div class="form-group">
          <label for="register__email">Email</label>
          <input type="email" class="form-control" id="register__email" ng-model="vm.email" placeholder="ex. john@notgoogle.com" />
        </div>

        <div class="form-group">
          <label for="register__username">Username</label>
          <input type="text" class="form-control" id="register__username" ng-model="vm.username" placeholder="ex. john" />
        </div>

        <div class="form-group">
          <label for="register__password">Password</label>
          <input type="password" class="form-control" id="register__password" ng-model="vm.password" placeholder="ex. thisisnotgoogleplus" />
        </div>

        <div class="form-group">
          <button type="submit" class="btn btn-primary">Submit</button>
        </div>
      </form>
    </div>
  </div>
</div>
```

На этот раз мы не будем вдаваться в подробности, поскольку это довольно простая HTML разметка. Большинство классов взято из Bootstrap, который включен в шаблонный проект. Стоит обратить внимание только на две строки:

```html
<form role="form" ng-submit="vm.register()">
```

Эта строка отвечает за вызов функции `$scope.register`, которую мы создали в нашем контроллере. `ng-submit` будет вызывать `$scope.register` при отправке формы. Если Вы сталкивались с Angular раньше, то уже наверное использовали `$scope`. В этом учебном пособии мы стараемся избегать использования `$scope`, где это возможно, используя вместо него `vm` - сокращение от ViewModel. Чтобы узнать больше об этом, смотри раздел [Контроллеры](https://github.com/johnpapa/angularjs-styleguide#controllers) руководства по стилю кодирования AngularJS от John Papa.

```html
<input type="email" class="form-control" id="register__email" ng-model="vm.email" placeholder="ex. john@notgoogle.com" />
```

В каждом `<input />` Вы увидите ещё одну директиву - `ng-model`. `ng-model` отвечает за сохранение введенного значения в ViewModel. Используя эту директиву мы передаём значения имени пользователя, пароля и электронной почты в ViewModel при вызове `vm.register`.

## Управляем интерфейсом с помощью RegisterController

После того как написаны служба и интерфейс, нам необходим контроллер, чтобы связать их. Разрабатываемый нами `RegisterController` контроллер позволит вызывать метод `register` службы `Authentication` каждый раз, когда пользователь отправляет только что созданную форму.

Создайте файл в каталоге `static/javascripts/authentication/controllers/` с названием `register.controller.js` и добавьте в него следующий код:

```javascript
/**
* Register controller
* @namespace thinkster.authentication.controllers
*/
(function () {
  'use strict';

  angular
    .module('thinkster.authentication.controllers')
    .controller('RegisterController', RegisterController);

  RegisterController.$inject = ['$location', '$scope', 'Authentication'];

  /**
  * @namespace RegisterController
  */
  function RegisterController($location, $scope, Authentication) {
    var vm = this;

    vm.register = register;

    /**
    * @name register
    * @desc Register a new user
    * @memberOf thinkster.authentication.controllers.RegisterController
    */
    function register() {
      Authentication.register(vm.email, vm.password, vm.username);
    }
  }
})();
```

Как обычно мы пропустим уже известные нам понятия и поговорим о новых.

```javascript
.controller('RegisterController', RegisterController);
```

Этот фрагмент кода похож на тот, где мы регистрировали нашу службу. Разница заключается в том, что в этот раз мы регистрируем контроллер.

`vm` позволяет только что созданному шаблону получить доступ к методу `register`, который мы определим позднее в контроллере.

```javascript
Authentication.register(vm.email, vm.password, vm.username);
```

Здесь мы вызываем службу, созданную несколько минут назад. Мы передаём имя пользователя, пароль и электронную почту из `vm`.

## Регистрация маршрутов и модулей

Давайте настроим маршрутизацию на стороне клиента, чтобы пользователи приложения могли перейти к форме для регистрации.

Создайте файл `thinkster.routes.js` в каталоге `static/javascripts` и добавьте в него следующий код:

```javascript
(function () {
  'use strict';

  angular
    .module('thinkster.routes')
    .config(config);

  config.$inject = ['$routeProvider'];

  /**
  * @name config
  * @desc Define valid application routes
  */
  function config($routeProvider) {
    $routeProvider.when('/register', {
      controller: 'RegisterController', 
      controllerAs: 'vm',
      templateUrl: '/static/templates/authentication/register.html'
    }).otherwise('/');
  }
})();
```

Необходимо коснуться нескольких моментов в этом фрагменте кода.

```javascript
.config(config);
```

Angular, как и практически любой другой фреймворк, позволяет Вам редактировать его настройки. Вы можете сделать с помощью `.config`.

```javascript
function config($routeProvider) {
```

Здесь мы внедряем зависимость `$routeProvider`, что позволяет нам добавить маршрутизацию в клиентское приложение.

```javascript
$routeProvider.when('/register', {
```

`$routeProvider.when` принимает два аргумента: путь и объект с настройками. Здесь мы используем `/register` в качестве пути, поскольку хотим, чтобы после перехода по этому пути отображалась регистрационная форма.

```javascript
controller: 'RegisterController',
controllerAs: 'vm',
```

Один из ключей, который Вы можете добавить в объект с настройками - это `controller`. Это позволяет сопоставить определенный контроллер с этим маршрутом. Здесь мы используем `RegisterController` контроллер, созданный нами ранее. `controllerAs` - это другой настраиваемый параметр. Он необходим, чтобы мы могли использовать переменную `vm`. Короче говоря, что хотим использовать псевдоним `vm`, ссылаясь на контроллер в шаблоне.

```javascript
templateUrl: '/static/templates/authentication/register.html'
```

Другой ключ, используемый нами - это `templateUrl`. `templateUrl` содержит URL строку с адресом, где находится шаблон, который мы хотим использовать для этого маршрута.

```javascript
}).otherwise('/');
```

Мы добавим дополнительные маршруты в дальнейшем, но может так получиться, что пользователь введет URL адрес, не поддерживаемый нашим приложением. В этом случае `$routeProvider.otherwise` перенаправит пользователя по указанному пути; в данном случае - '/'.

## Настройка модулей в AngularJS

Давайте кратко рассмотрим использование модулей в AngulaJS.

В Angular Вы должны определить модули перед тем как их использовать. Таким образом, нам надо определить `thinkster.authentication.services`, `thinkster.authentication.controllers` и `thinkster.routes`. Поскольку `thinkster.authentication.services` и `thinkster.authentication.controllers` являются подмодулями `thinkster.authentication` нам необходимо также создать модуль `thinkster.authentication`.

Создайте файл `authentication.module.js` в каталоге `static/javascripts/authentication/` и добавьте в него следующий код:

```javascript
(function () {
  'use strict';

  angular
    .module('thinkster.authentication', [
      'thinkster.authentication.controllers',
      'thinkster.authentication.services'
    ]);

  angular
    .module('thinkster.authentication.controllers', []);

  angular
    .module('thinkster.authentication.services', ['ngCookies']);
})();
```

Здесь есть несколько интересных моментов, на которых стоит остановиться.

```javascript
angular
  .module('thinkster.authentication', [
    'thinkster.authentication.controllers',
    'thinkster.authentication.services'
  ]);
```

Этот синтаксис определяет модуль `thinkster.authentication` со службами `thinkster.authentication.controllers` и `thinkster.authentication.services` в качестве зависимостей.

```javascript
angular
  .module('thinkster.authentication.controllers', []);
```

Этот синтаксис определяет модуль `thinkster.authentication.controllers` без зависимостей.

Теперь нам необходимо добавить `thinkster.authentication` и `thinkster.routes` в качестве зависимостей в `thinkster`.

Откройте файл `static/javascripts/thinkster.js`, определите требуемые модули и добавьте их в качестве зависимостей в модуль `thinkster`. Обратите внимание, что `thinkster.routes` использует `ngRoute`, который добавлен в шаблонный проект.

```javascript
(function () {
  'use strict';

  angular
    .module('thinkster', [
      'thinkster.routes',
      'thinkster.authentication'
    ]);

  angular
    .module('thinkster.routes', ['ngRoute']);
})();
```

## Маршутизация с помощью `#`

По умолчанию Angular использует маршрутизацию с помощью хэшей `#`. Если Вы когда-нибудь видели URL адрес похожий на `www.google.com/#/search`, то Вы понимаете о чём я говорю. Лично я считаю такие URL адреса уродливыми. Чтобы избавиться от маршрутизации с помощью хэшей, мы можем включить `$locationProvider.html5Mode`. В старых браузерах, которые не поддерживают HTML5 маршрутизацию, Angular, определив отсутствие поддержки, будет использовать маршрутизацию с помощью хэшей.

Создайте в каталоге `static/javascripts/` файл `thinkster.config.js` и добавьте в него следующий код:

```javascript
(function () {
  'use strict';

  angular
    .module('thinkster.config')
    .config(config);

  config.$inject = ['$locationProvider'];

  /**
  * @name config
  * @desc Enable HTML5 routing
  */
  function config($locationProvider) {
    $locationProvider.html5Mode(true);
    $locationProvider.hashPrefix('!');
  }
})();
```

Как было сказано ранее, включение `$locationProvider.html5Mode` позволяет избавится от знака `#` в URL адресах. Параметр `$locationProvider.hashPrefix` преобразует `#` в `!#`. Эта настройка в основном нужна для улучшения работы поисковых систем.

Поскольку здесь мы используем новый модуль, нам нужно открыть файл `static/javascripts/thinkster.js`, определить модуль и добавить его в качестве зависимости в модуль `thinkster`.

```javascript
angular
  .module('thinkster', [
    'thinkster.config',
    // ...
  ]);

angular
  .module('thinkster.config', []);
```

## Добавляем новые .js файлы

В этой главе мы создали несколько новых JavaScript файлов. Мы должны добавить их в клиентское приложение, дописав их в файл `templates/javascripts.html` внутри блока `{% compress js %}`.

Откройте `templates/javascripts.html` и добавьте следующий код перед тегом `{% endcompress %}`:

```html
<script type="text/javascript" src="{% static 'javascripts/thinkster.config.js' %}"></script>
<script type="text/javascript" src="{% static 'javascripts/thinkster.routes.js' %}"></script>
<script type="text/javascript" src="{% static 'javascripts/authentication/authentication.module.js' %}"></script>
<script type="text/javascript" src="{% static 'javascripts/authentication/services/authentication.service.js' %}"></script>
<script type="text/javascript" src="{% static 'javascripts/authentication/controllers/register.controller.js' %}"></script>
```

## Осуществляем защиту от СSRF атак

Поскольку мы используем аутентификацию, основанную на сессиях, нам нужно позаботиться о CSRF защите. Мы не будем вдаваться в подробности, связанные с CSRF здесь, поскольку это выходит за рамки этого пособия, но достаточно сказать, что CSRF - это очень плохо.

Django по умолчанию хранит CSRF токен в куки с названием `csrftoken` и ожидает получить заголовок с именем `X-CSRFToken` для любого опасного HTTP запроса (`POST`, `PUT`, `PATCH`, `DELETE`). Мы можем легко настроить Angular для создания такого заголовка.

Откройте файл `static/javascripts/thinkster.js` и добавьте следующий код после определения Ваших модулей:

```javascript
angular
  .module('thinkster')
  .run(run);

run.$inject = ['$http'];

/**
* @name run
* @desc Update xsrf $http headers to align with Django's defaults
*/
function run($http) {
  $http.defaults.xsrfHeaderName = 'X-CSRFToken';
  $http.defaults.xsrfCookieName = 'csrftoken';
}
```
# Контрольная точка

Попытайтесь зарегистрировать нового пользователя, запустив сервер (`python manage.py runserver`), перейдя по адресу `http://localhost:8000/register` в Вашем браузере и заполнив форму.

Если регистрация сработала правильно, то Вы сможете просмотреть созданный таким образом объект `Account`, открыв в терминале оболочку (`python manage.py shell`) и выполнив следующие команды:

```
>>> from authentication.models import Account
>>> Account.objects.latest('created_at')
```

Информация внутри возвращеного таким образом объекта `Account` должен совпадать с той, который Вы только что ввели, заполнив регистрационную форму.
