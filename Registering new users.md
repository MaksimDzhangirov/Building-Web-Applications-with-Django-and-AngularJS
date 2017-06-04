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

