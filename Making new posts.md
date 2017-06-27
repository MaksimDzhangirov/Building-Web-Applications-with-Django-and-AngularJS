# Создаём новые посты

Учитывая, что у нас уже есть необходимые конечные точки, следующий шагом, необходимым для того, чтобы пользователи могли создавать новые посты, является интерфейс. Для этого мы добавим кнопку в нижнем правом углу экрана. При нажатии на эту кнопку появляется модальное окно, в котором пользователь должен ввести свой пост.

Пока мы хотим, чтобы эта кнопка отображалась только на главной странице, поэтому откройте `static/templates/layout/index.html` и добавьте следующий фрагмент кода в конец файла:

```html
<a class="btn btn-primary btn-fab btn-raised mdi-content-add btn-add-new-post"
  href="javascript:void(0)"
  ng-show="vm.isAuthenticated"
  ng-dialog="/static/templates/posts/new-post.html"
  ng-dialog-controller="NewPostController as vm"></a>
```

Тег `a` в этом фрагменте использует директиву `ngDialog`, добавленную в качестве зависимости ранее, чтобы отображать модальные окна каждый раз, когда пользователь хочет создать новый пост.

Поскольку мы хотим, чтобы кнопка была зафиксирована в нижнем правом углу экрана, нам необходимо также добавить новое CSS правило.

Откройте `static/stylesheets/styles.css` и добавьте это правило в конец файла:

```css
.btn-add-new-post {
  position: fixed;
  bottom: 20px;
  right: 20px;
}
```

## Интерфейс для отправки новых постов

Теперь нам нужно создать форму, в которую пользователь будет вводить свой новый пост. Откройте `static/templates/posts/new-post.html` и добавьте следующий код в конец файла:

```html
<form role="form" ng-submit="vm.submit()">
  <div class="form-group">
    <label for="post__content">New Post</label>
    <textarea class="form-control" 
              id="post__content" 
              rows="3" 
              placeholder="ex. This is my first time posting on Not Google Plus!" 
              ng-model="vm.content">
    </textarea>
  </div>

  <div class="form-group">
    <button type="submit" class="btn btn-primary">
      Submit
    </button>
  </div>
</form>
```

## Управляем интерфейсом для нового поста с помощью NewPostController

Создайте файл `static/javascripts/posts/controllers/new-post.controller.js` со следующим содержимым:

```javascript
/**
* NewPostController
* @namespace thinkster.posts.controllers
*/
(function () {
  'use strict';

  angular
    .module('thinkster.posts.controllers')
    .controller('NewPostController', NewPostController);

  NewPostController.$inject = ['$rootScope', '$scope', 'Authentication', 'Snackbar', 'Posts'];

  /**
  * @namespace NewPostController
  */
  function NewPostController($rootScope, $scope, Authentication, Snackbar, Posts) {
    var vm = this;

    vm.submit = submit;

    /**
    * @name submit
    * @desc Create a new Post
    * @memberOf thinkster.posts.controllers.NewPostController
    */
    function submit() {
      $rootScope.$broadcast('post.created', {
        content: vm.content,
        author: {
          username: Authentication.getAuthenticatedAccount().username
        }
      });

      $scope.closeThisDialog();

      Posts.create(vm.content).then(createPostSuccessFn, createPostErrorFn);


      /**
      * @name createPostSuccessFn
      * @desc Show snackbar with success message
      */
      function createPostSuccessFn(data, status, headers, config) {
        Snackbar.show('Success! Post created.');
      }


      /**
      * @name createPostErrorFn
      * @desc Propogate error event and show snackbar with error message
      */
      function createPostErrorFn(data, status, headers, config) {
        $rootScope.$broadcast('post.created.error');
        Snackbar.error(data.error);
      }
    }
  }
})();
```

В этом фрагменте кода есть несколько частей, о которых мы должны упомянуть.

```javascript
$rootScope.$broadcast('post.created', {
  content: $scope.content,
  author: {
    username: Authentication.getAuthenticatedAccount().username
  }
});
```

Ранее мы настроили перехватчик событий в IndexController, который отслеживал событие `post.created` и затем помещал новый пост в начало массива `vm.posts`. Давайте рассмотрим этот момент подробнее, поскольку, как оказывается, он является важной особенностью больших веб-приложений.

Здесь мы действуем *оптимистично*, предполагая, что ответ API для `Posts.create()` будет содержать код статуса 200, сообщающий, что пост успешно создан, как и планировалось. Сначала это может показаться плохой идеей. Во время запроса что-то может пойди не так и тогда наши данные в `vm.posts` перестанут соответствовать действительности. Почему бы нам просто не подождать ответа от сервера?

Когда я говорил, что мы увеличиваем производительность с точки зрения восприятия пользователем, я говорил о следующем. Мы хотим, чтобы с точки зрения пользователя ответ от сервера приходил мгновенно.

Дело в том, что этот запрос редко завершается с ошибкой. Существует только два случая, когда такое будет происходить в реальном приложении: пользователь не аутентифицирован или не доступен сервер.
