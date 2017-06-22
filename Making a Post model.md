# Создаём модель Post

В этом разделе мы разработаем новое приложение и создадим `Post` модель, которая будет подобна статусу на Facebook или твиту в Tweeter. После создания нашей модели мы перейдём к сериализации `Post`ов, а затем добавим несколько новых конечных точек в наше API.

## Создаём приложение `posts`

Прежде всего создайте новое приложение под названием `posts`.

```
python manage.py startapp posts
```

Помните, что каждый раз, когда Вы создаёте новое приложение, Вы должны добавить его в настройку `INSTALLED_APPS`. Откройте файл `thinkster_django_angular_boilerplate/settings.py` и измените его следующим образом:

```python
INSTALLED_APPS = (
    # ...
    'posts',
)
```

## Создаём модель Post

При создании приложения `posts`, Django добавил в него новый файл под названием `posts/models.py`. Откройте его и добавьте следующие содержимое:

```python
from django.db import models
from authentication.models import Account

class Post(models.Model):
    author = models.ForeignKey(Account)
    content = models.TextField()

    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __unicode__(self):
        return '{0}'.format(self.content)
```

Наш метод построчного изучения кода пока что не подводил нас. Почему бы не воспользоваться им опять? Давайте начнем.

```python
author = models.ForeignKey(Account)
```

Поскольку каждый пользователь может иметь множество объектов `Post`, нам нужно создать отношение один-ко-многим.

В Django это можно сделать с помощью поля `ForeignKey`, связав каждый `Post` с `Account`.

Django понимает, что внешний ключ, который мы использовали здесь, должен также создавать обратное отношение. То есть, для заданного объекта `Account`, Вы всегда должны иметь возможность получить `Post`ы, связанные с этим пользователем. В Django эти объекты `Post` доступны следующим образом `Account.post_set` (а не `Account.posts`).

Теперь, когда модель существует, не забудьте осуществить миграции.

```
$ python manage.py makemigrations
$ python manage.py migrate
```

## Сериализуем модель Post

Создайте новый файл `serializers.py` в каталоге `posts/` и добавьте в него следующий код:

```python
from rest_framework import serializers

from authentication.serializers import Account
from posts.models import Post


class PostSerializer(serializers.ModelSerializer):
    author = AccountSerializer(read_only=True, required=False)

    class Meta:
        model = Post

        fields = ('id', 'author', 'content', 'created_at', 'updated_at')
        read_only_fields = ('id', 'created_at', 'updated_at')

    def get_validation_exclusions(self, *args, **kwargs):
        exclusions = super(PostSerializer, self).get_validation_exclusions()

        return exclusions + ['author']
```

В этом фрагменте кода используются уже рассмотренные нами понятия, поэтому остановимся только на одной строке, на которую я хотел бы обратить внимание.

```python
author = AccountSerializer(read_only=True, required=False)
```

Раньше мы явно перечисляли названия полей в нашем `AccountSerializer`, но здесь определили его немного по-другому.

При сериализации объекта `Post`, мы хотим добавить в него всю информацию об авторе. В Django REST фреймворке это называется вложенным отношением. Проще говоря, мы сериализуем `Account`, связанный с этим `Post` и добавляем его в наш JSON.

Мы передаём параметр `read_only=True`, поскольку мы не будем обновлять объект `Account` с помощью `PostSerializer`. Мы также присваиваем здесь `required=False`, поскольку мы автоматически присвоим автора каждому посту.

```python
def get_validation_exclusions(self, *args, **kwargs):
    exclusions = super(PostSerializer, self).get_validation_exclusions()

    return exclusions + ['author']
```

По той же причине, по которой мы использовали `required=False`, мы должны также добавить `author` в список полей, которые не будут проверяться валидатором.

## Создаём представления для API Post объектов

Следующий шаг при создании Post объектов - это добавление конечной точки API, которая будет осуществлять различные операции с `Post` моделью, такие как создание или обновление.

Замените содержимое `posts/views.py` на следующее:

```python
from rest_framework import permissions, viewsets
from rest_framework.response import Response

from posts.models import Post
from posts.permissions import IsAuthorOfPost
from posts.serializers import PostSerializer


class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.order_by('-created_at')
    serializer_class = PostSerializer

    def get_permissions(self):
        if self.request.method in permissions.SAFE_METHODS:
            return (permissions.AllowAny(),)
        return (permissions.IsAuthenticated(), IsAuthorOfPost(),)

def perform_create(self, serializer):
    instance = serializer.save(author=self.request.user)

    return super(PostViewSet, self).perform_create(serializer)



class AccountPostsViewSet(viewsets.ViewSet):
    queryset = Post.objects.select_related('author').all()
    serializer_class = PostSerializer

    def list(self, request, account_username=None):
        queryset = self.queryset.filter(author__username=account_username)
        serializer = self.serializer_class(queryset, many=True)

        return Response(serializer.data)
```

Не кажутся ли Вам эти представления знакомыми? Они не сильно отличаются от тех, которые мы использовали при создании объектов `Users`.

```python
def perform_create(self, serializer):
    instance = serializer.save(author=self.request.user)

    return super(PostViewSet, self).perform_create(serializer)
```

Метод `perform_create` вызывается перед тем как сохраняется модель этого представления.

Когда создаётся объект `Post`, его нужно связать с автором. Требовать от автора вводить своё имя или id при добавлении поста на сайт - это плохая практика, поэтому мы реализуем эту связь за него с помощью хука `perform_create`. Мы просто извлекаем информацию о пользователе, связанным с данным запросом, и делаем его автором этого `Post`.

```python
def get_permissions(self):
    if self.request.method in permissions.SAFE_METHODS:
        return (permissions.AllowAny(),)
    return (permissions.IsAuthenticated(), IsAuthorOfPost(),)
```

Подобно тем правам доступа, которые мы использовали для viewset `Account`, здесь для опасных HTTP методов мы требуем, чтобы пользователь был аутентифицирован и авторизован, если хочет изменить этот `Post`. Чуть ниже мы создадим право доступа `IsAuthorOfPost`. Если HTTP метод безопасен, мы разрешаем любому пользователю доступ к этому представлению.

```python
class AccountPostsViewSet(viewsets.ViewSet):
```

Этот viewset будет использоваться для вывода списка постов, связанных с конкретным `Account`.

```python
queryset = self.queryset.filter(author__username=account_username)
```

Здесь мы фильтруем наш запрос, основываясь на имени автора. Аргумент `account_username` извлекается из URL адреса, шаблон которого мы вскоре создадим.

## Создаём право доступа IsAuthorOfPost

Создайте файл `permissions.py` в каталоге `posts/` со следующим содержимым:

```python
from rest_framework import permissions


class IsAuthorOfPost(permissions.BasePermission):
    def has_object_permission(self, request, view, post):
        if request.user:
            return post.author == request.user
        return False
```

Мы не будем комментировать этот фрагмент кода. Это право доступа практически идентично тому, которое мы создали ранее.

## Создаём конечную точку API для постов

После создания представлений, пора добавить конечные точки в наше API.

Откройте `thinkster_django_angular_boilerplate/urls.py` и добавьте следующий импорт:

```python
from posts.views import AccountPostsViewSet, PostViewSet
```

Теперь добавьте эти строки сразу после `urlpatterns = patterns(`:

```python
router.register(r'posts', PostViewSet)

accounts_router = routers.NestedSimpleRouter(
    router, r'accounts', lookup='account'
)
accounts_router.register(r'posts', AccountPostsViewSet)
```

`accounts_router` позволяет реализовать вложенную маршрутизацию, необходимую для доступа к постам определенного автора. Теперь Вам нужно добавить `accounts_router` в `urlpatterns` как показано ниже:

```python
urlpatterns = patterns(
  # ...

  url(r'^api/v1/', include(router.urls)),
  url(r'^api/v1/', include(accounts_router.urls)),

  # ...
)
```

## Контрольная точка

Теперь откройте оболочку в терминале с помощью команды `python manage.py shell` и поэкспериментируйте с созданием и сериализацией `Post` объектов.

```
>>> from authentication.models import Account
>>> from posts.models import Post
>>> from posts.serializers import PostSerializer
>>> account = Account.objects.latest('created_at')
>>> post = Post.objects.create(author=account, content='I promise this is not Google Plus!')
>>> serialized_post = PostSerializer(post)
>>> serialized_post.data
```

Мы убедимся в правильной работе представлений в конце следующего раздела.
