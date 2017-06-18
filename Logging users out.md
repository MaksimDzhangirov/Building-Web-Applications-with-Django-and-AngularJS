# Реализация возможности выхода пользователей из системы

Учитывая то, что пользователи могут зарегистрироваться и войти в систему, логично предположить, что они хотели бы также иметь способ выйти из системы. ЛЮди могут начать негодовать,если не смогут выйти из системы.

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

