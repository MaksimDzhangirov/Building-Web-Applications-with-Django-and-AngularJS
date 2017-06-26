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

## Упрпавляем интерфейсом для нового поста с помощью NewPostController
