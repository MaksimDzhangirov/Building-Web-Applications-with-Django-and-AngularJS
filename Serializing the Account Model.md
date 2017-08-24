# Сериализуем модель Account

AngularJS приложение, которое мы собираемся создать, посылает AJAX запросы на сервер, чтобы получить данные, затем используемые для отображения пользователю. Прежде чем мы отправим данные обратно клиенту, нам необходимо привести их к формату понятному клиенту; в нашем случае мы выбираем JSON. Процесс преобразования Django моделей в JSON называется сериализацией и о ней мы будем говорить в этом разделе.

Поскольку модель, которую мы хотим сериализовать, называется `Account`, создаваемый нами сериализатор будет называться `AccountSerializer`.

## Django REST фреймворк

В шаблонный проект, который Вы клонировали ранее, мы добавили проект под названием Django REST фреймворк. Django REST фреймворк - это набор инструментов, предоставляющий доступ к определенному функционалу часто используемому в большинстве веб приложений, включая сериализаторы. Мы будем использовать этот функционал в нашем учебном пособии, чтобы сьэкономить время и сберечь нервы. Сейчас мы начнем наше первое знакомство с Django REST фреймворком.

## AccountSerializer

Прежде, чем написать наши сериализаторы, давайте создадим файл `serializers.py` внутри нашего приложения `authentication`:

```
$ touch authentication/serializers.py
```

Откройте файл `authentication/serializers.py` и добавьте следующий код и импорты:

```python
from django.contrib.auth import update_session_auth_hash

from rest_framework import serializers

from authentication.models import Account


class AccountSerializer(serializers.ModelSerializer):
    password = serializers.CharField(write_only=True, required=False)
    confirm_password = serializers.CharField(write_only=True, required=False)

    class Meta:
        model = Account
        fields = ('id', 'email', 'username', 'created_at', 'updated_at',
                  'first_name', 'last_name', 'tagline', 'password',
                  'confirm_password',)
        read_only_fields = ('created_at', 'updated_at',)

        def create(self, validated_data):
            return Account.objects.create(**validated_data)

        def update(self, instance, validated_data):
            instance.username = validated_data.get('username', instance.username)
            instance.tagline = validated_data.get('tagline', instance.tagline)

            instance.save()

            password = validated_data.get('password', None)
            confirm_password = validated_data.get('confirm_password', None)

            if password and confirm_password and password == confirm_password:
                instance.set_password(password)
                instance.save()

            update_session_auth_hash(self.context.get('request'), instance)

            return instance
```

> Замечание. С этого момента мы будем объявлять импорты, которые используются, в каждом фрагменте кода. Они могут уже присутствовать в файле. В этом случае их не нужно добавлять в него второй раз.

Давайте рассмотрим код подробнее.

```python
password = serializers.CharField(write_only=True, required=False)
confirm_password = serializers.CharField(write_only=True, required=False)
```

Вместо того, чтобы добавить поле `password` в кортеж `fields`, о котором мы поговорим немного позднее, мы явно определим поле в начале класса `AccountSerializer`. Это связано с тем, что объявляя поле таким образом, мы можем передать в него аргумент `required=False`. Каждое поле в кортеже `fields` является обязательным для заполнения, но мы не хотим, чтобы пользователю нужно было каждый раз вводить новый пароль при обновлении учетной записи (поскольку поле `password` являлось бы обязательным для заполнения - прим. переводчика), если только он сам не захочет сменить его.

Поле `confirm_pssword` определяется подобно `password` и используется только для того, чтобы гарантировать, что пользователь случайно не совершил опечатку при вводе пароля.

Также обратите внимание на использование аргумента `write_only=True`. Пароль пользователя, даже после хэширования и добавления соли, не должен быть доступен клиенту в AJAX ответе.

```python
class Meta:
```

Подкласс `Meta` определяет метаданные, требуемые для работы сериализатора. В нем мы определили несколько часто задаваемых для класса класса `Meta` атрибутов .

```python
model = Account
```

Поскольку этот сериализатор наследуется от класса `serializers.ModelSerializer`, очевидно, что мы должны указать ему какую модель нужно сериализовать. Указание модели гарантирует, что могут быть сериализованы только атрибуты этой модели или явно создаваемые поля. Сейчас мы рассмотрим сериализацию атрибутов модели, а явно создаваемые поля немного позднее.

```python
fields = ('id', 'email', 'username', 'created_at', 'updated_at',
          'first_name', 'last_name', 'tagline', 'password',
          'confirm_password',)
```

В атрибуте `fields` класса `Meta` указываются какие атрибуты модели `Account` должны быть сериализованы. Необходимо быть осторожными при указании полей для сериализации, поскольку некоторые поля, например, `is_superuser`, не должны быть доступны клиенту по соображениям безопасности.

```python
read_only_fields = ('created_at', 'updated_at',)
```

Если Вы помните, при создании модели `Account`, мы сделали поля `created_at` и `updated_at` самообновляемыми. Из-за этого мы добавим их к списку полей, которые должны быть доступны только для чтения.

```python
def create(self, validated_data):
    # ...

def update(self, instance, validated_data):
    # ...
```

Ранее мы упоминали, что иногда мы хотим преобразовать JSON в Python объект. Этот процесс называется десериализацией и осуществляется методами `.create()` и `.update()`. При создании нового объекта, например, `Account`, используется `.create()`. Затем при обновлении  этой модели `Account`, будет использоваться `.update()`.

```python
instance.username = attrs.get('username', instance.username)
instance.tagline = attrs.get('tagline', instance.tagline)
```

Мы позволим пользователю обновлять свои атрибуты `username` (имя пользователя) и `tagline` (девиз). Если эти ключи присутствуют в словаре, мы будем использовать новое значение. В противном случае используется текущее значение объекта `instance`. Здесь `instance` имеет тип `Account`.

```python
password = validated_data.get('password', None)
confirm_password = validated_data.get('confirm_password', None)

if password and confirm_password and password == confirm_password:
    instance.set_password(password)
    instance.save()
```

Прежде чем обновить пароль пользователя, нам необходимо убедиться, что оба поля `password` и `password_confirmation` были заполнены. Затем мы проверяем содержат ли эти поля одинаковые значения.

Убедившись в необходимости обновления пароля, мы должны использовать `Account.set_password()` для осуществления обновления. `Account.set_password()` используется для сохранения паролей, защищая их от взлома. Важно отметить, что мы должны явно сохранить модель после обновления пароля.

> Это простейшая реализация того как следует проверять пароль. Я не рекомендую использовать её в реальных системах, но для наших учебных целей она прекрасно подходит.

```python
update_session_auth_hash(self.context.get('request'), instance)
```

При обновлении пароля пользователя должен быть также явно обновлен его хэш проверки подлинности сессии. Если не сделать этого, пользователь не будет аутентифицирован при следующем запросе и ему придется снова входить в систему.

# Контрольная точка

Теперь у нас не должно возникнуть проблем с JSON сериализацией объекта `Account`. Опять откройте оболочку Django, выполнив команду `python manage.py shell` и попытайтесь ввести следующие команды:

```
>>> from authentication.models import Account
>>> from authentication.serializers import AccountSerializer
>>> account = Account.objects.latest('created_at')
>>> serialized_account = AccountSerializer(account)
>>> serialized_account.data.get('email')
>>> serialized_account.data.get('username')
```

