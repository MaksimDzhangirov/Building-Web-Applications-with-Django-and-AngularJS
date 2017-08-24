# Расширяем встроенную в Django модель User

В Django существует встроенная модель `User`, которая включает в себя множество функций доступных по умолчанию. Проблема заключается в том, что эта модель не может быть расширена для добавления дополнительной информации. Например, мы хотим, чтобы каждый пользователь имел дополнительное поле `tagline` (девиз, слоган), который бы отображался в их профиле. У модели `User` нет такого атрибута и мы не можем добавить его в неё.

Модель `User` наследуется от `AbstractBaseUser`. Именно из неё модель `User` наследует большую часть своего функционала. Создавая новую модель `Account` и наследуя её от `AbstractBaseUser`, мы получаем необходимый функционал модели `User` (хеширование паролей, управление сессиями и т. д.) и возможность расширять модель `Account` для добавления дополнительной информации, например, поля `tagline`.

В Django понятие "приложение" используется для эффективной организации кода. Приложение - это модуль, который содержит код моделей, представлений, сериализаторов и т. д., связанных друг с другом определенным образом. Например, нашим первым шагом в реализации нашего Django и AngularJS веб приложения будет создание приложения под названием `authentication`. Приложение `authentication` будет содержать код, относящийся к модели `Account`, о которой мы только что говорили, а также представления для входа в систему, выхода из системы и регистрации.

Создайте новое приложение `authentication`, выполнив следующую команду:
```
$ python manage.py startapp authentication
```

## Создание модели Account

Для начала создадим модель `Account`, о которой мы только что говорили.

Откройте файл `authentication/models.py` в своём любимом текстовом редакторе и отредактируйте его, чтобы он выглядел следующим образом:

```python
from django.contrib.auth.models import AbstractBaseUser
from django.db import models

class Account(AbstractBaseUser):
    email = models.EmailField(unique=True)
    username = models.CharField(max_length=40, unique=True)

    first_name = models.CharField(max_length=40, blank=True)
    last_name = models.CharField(max_length=40, blank=True)
    tagline = models.CharField(max_length=140, blank=True)

    is_admin = models.BooleanField(default=False)

    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    objects = AccountManager()

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username']

    def __unicode__(self):
        return self.email

    def get_full_name(self):
        return ' '.join([self.first_name, self.last_name])

    def get_short_name(self):
        return self.first_name

```
Давайте подробно рассмотрим каждый атрибут и метод.
```python
email = models.EmailField(unique=True)

# ...

USERNAME_FIELD = 'email'
```
Встроенная в Django модель `User` использует в качество логина имя пользователя. В отличие от неё для этой же цели наше приложение будет использовать адрес электронной почты пользователя.

Чтобы указать Django, чтобы мы хотим использовать электронную почту в качестве имени пользователя для этой модели, мы присвоим атрибуту `USERNAME_FIELD` поле `email`. Поле, указанное в `USERNAME_FIELD` должно быть уникально, поэтому мы передаем аргумент `unique=True` при определении поля `email`.

```python
username = models.CharField(max_length=40, unique=True)
```
Несмотря на то, что мы будем входить в систему, используя электронную почту, мы все равно хотим, чтобы у каждого пользователя было имя. Оно нам потребуется при отображении постов пользователя и на его странице профиля. Мы также будем использовать имя пользователя в наших URL, поэтому имя пользователя должно быть уникально. Поэтому мы аргумент `unique=True` в поле `username`.
```python
first_name = models.CharField(max_length=40, blank=True)
last_name = models.CharField(max_length=40, blank=True)
```
В идеале мы хотели бы иметь возможность обращаться к нашим пользователям по имени и фамилии. Поскольку мы понимаем, что не все пользователи хотят выдавать свои личные данные, мы делаем поля `first_name ` и `last_name ` необязательными для заполнения, передавая в них в качестве аргумента `blank=True`.

```python
tagline = models.CharField(max_length=140, blank=True)
```

Как было сказано ранее, атрибут `tagline` будет отображаться в профиле пользователя. Находясь там, он позволяет больше узнать о личности пользователя, поэтому его стоит добавить.

```python
created_at = models.DateTimeField(auto_now_add=True)
updated_at = models.DateTimeField(auto_now=True)
```
В поле `created_at` записывается время создания объекта `Account`. Передавая `auto_now_add=True` в `models.DateTimeField`, мы указываем Django, что значение должны автоматически устанавливатся при создании объекта и не изменяется после этого.

Подобно `created_at` Django автоматически изменяет поле `updated_at`. Отличие между `auto_now_add=True` и `auto_now=True` заключается в том, что аргумент `auto_now=True` приводит к тому, что поле обновляется каждый раз при сохранении объекта.

```python
objects = AccountManager()
```

Когда Вы хотите получить экземпляр модели в Django, Вы используете выражение вида `Model.objects.get(**kwargs)`. Здесь атрибут `objects` является классом `Manager`, который принято называть следующим образом `<название модели>Manager`. В нашем случае мы создадим класс `AccountManager`. Мы сделаем это сразу же в следующем разделе.

```python
REQUIRED_FIELDS = ['username']
```
Мы будем отображать имя пользователя в нескольких местах. Таким образом, оно обязательно для заполнения, поэтому мы включаем его в список `REQUIRED_FIELDS`. Обычно для этого достаточно аргумента `required=True`, но поскольку эта модель заменяет модель `User`, Django требует, чтобы мы указали обязательные поля таким образом.

```python
def __unicode__(self):
    return self.email
```   

При работе с моделями в командной строке, как мы вскоре увидим, стандартное представление объекта `Account` выглядит примерно следующим образом `<Account: Account>`. Поскольку мы будем работать с множеством различных учетных записей, такое представление не очень наглядно. Изменить это стандартное поведение можно переписав метод `__unicode__()`. В данном случае мы решили отображать вместо стандартного представления электронную почту пользователя. Строковое представление учетной записи с электронной почтой `james@notgoogleplus.com` теперь будет иметь вид `<Account: james@notgoogleplus.com>`.

```python
def get_full_name(self):
    return ' '.join([self.first_name, self.last_name])

def get_short_name(self):
    return self.first_name
```

Методы `get_full_name()` и `get_short_name()` добавлены соглаcно стандартам, принятым в Django. Мы не будем использовать ни один из этих методов, но все равно хорошей практикой будет добавление этих методов в соответствии со стандартами Django.

## Создаем класс Manager для модели Account

При использовании нестандартной модели для пользователя, также необходимо определить связанный с ней класс `Manager`, в котором будут описаны методы `create_user()` и `create_superuser()`.

Добавьте следующий класс перед определением класса `Account` в файл `authentication/models.py`:

```python
from django.contrib.auth.models import BaseUserManager

class AccountManager(BaseUserManager):
    def create_user(self, email, password=None, **kwargs):
        if not email:
            raise ValueError('Users must have a valid email address.')

        if not kwargs.get('username'):
            raise ValueError('Users must have a valid username.')

        account = self.model(
            email=self.normalize_email(email), username=kwargs.get('username')
        )

        account.set_password(password)
        account.save()

        return account

    def create_superuser(self, email, password, **kwargs):
        account = self.create_user(email, password, **kwargs)

        account.is_admin = True
        account.save()

        return account
```

Как и в случае с классом `Account`, давайте подробно рассмотрим каждую строчку в этом классе, комментируя только те части, которые нам не встречались ранее.

```python
if not email:
    raise ValueError('Users must have a valid email address.')

if not kwargs.get('username'):
    raise ValueError('Users must have a valid username.')
```

Поскольку у каждого пользователя должны быть заполнены поля электронной почты и имени, всякий раз должна возникать ошибка, если один из этих атрибутов отсутствует.

```python
account = self.model(
    email=self.normalize_email(email), username=kwargs.get('username')
)
```

Из-за того, что мы не определили атрибут `model` в классе `AccountManager`, `self.model` ссылается на атрибут `model` класса `BaseUserManager`. Это значение по умолчанию для настройки `settings.AUTH_USER_MODEL`, которую мы изменим совсем скоро, чтобы она указывала на класс `Account`.

```python
account = self.create_account(email, password, **kwargs)

account.is_admin = True
account.save()
```

Повторять один и тот же фрагмент кода - это плохая практика. Вместо того, чтобы копировать весь код из `create_account` и вставлять его в `create_superuser`, мы просто будем использовать `create_account` в качестве обработчика процесса создания пользователя во всех случаях. В этом случае в `create_superuser` необходимо будет реализовать только часть, требуемую для преобразования простой учетной записи `Account` в суперпользователя.

## Изменяем настройку Django AUTH_USER_MODEL

Несмотря на то, что мы создали модель `Account`, команда `python manage.py createsuperuser` (о которой мы поговорим немного позднее) все равно создаёт объекты, используя модель `User`. Это связано с тем, что на данный момент, Django считает, что мы по-прежнему хотим использовать модель `User` для аутентификации пользователя.

Чтобы исправить это и начать использовать модель `Account` в качестве нашей модели для аутентификации нам необходимо изменить настройку `settings.AUTH_USER_MODEL`.

Откройте файл `thinkster_django_angular_tutorial/settings.py` и добавьте следующую строку в его конец:

```python
AUTH_USER_MODEL = 'authentication.Account'
```

Это строка сообщает Django, что нужно использовать модель `Account` в приложении `authentication` для аутентификации пользователя.

## Устанавливаем Ваше первое приложение

В Django Вы должны явно указывать какие приложения хотите использовать. Поскольку мы ещё не добавили наше приложение `authentication` в список установленных приложений, сделаем это прямо сейчас.

Откройте файл `thinkster_django_angular_boilerplate/settings.py` и добавьте `authentication` в `INSTALLED_APPS` следующим образом:

```python
INSTALLED_APPS = (
    ...,
    'authentication',
)
```

## Осуществлям миграции для Вашего первого приложения

Релиз Django 1.7 стал настоящей манной небесной для разработчиков! Наконец-то стали доступны миграции!

Те, кто имел опыт работы с Rails, понятие миграций покажется знакомым. Если коротко, миграции осуществляют SQL запросы, необходимые для обновления структуры нашей базы данных вместо нас. В качестве примера рассмотрим только что созданную нами модель `Account`. Такие модели нужно хранить в базе данных, но в нашей базе до сих пор нет таблицы для хранения объектов `Account`. Что же делать? Мы создадим нашу первую миграцию! Миграция будет осуществлять добавление таблиц в базу и давать возможность откатить изменения, если мы совершили ошибку.

Как только будете готовы, создайте миграцию для приложения `authentication` и примените её:

```
$ python manage.py makemigrations
Migrations for 'authentication':
    0001_initial.py:
        - Create model Account
$ python manage.py migrate
Operations to perform:
    Synchronize unmigrated apps: rest_framework
    Apply all migrations: admin, authentication, contenttypes, auth, sessions
Synchronizing apps without migrations:
    Creating tables...
    Installing custom SQL...
    Installing indexes...
Running migrations:
    Applying authentication.0001_initial... OK
```

> Замечание. C этого момента мы не будем включать в материал то, что выводится в терминале при выполнении миграции.

## Создание суперпользователя

Давайте более подробно остановимся на команде `python manage.py createsuperuser`, о которой мы говорили чуть выше.

Разные пользователи имеют различные уровни доступа в любом конкретном приложении. Некоторые пользователи являются администраторами и имеют полный доступ, тогда как остальные являются обычными пользователями и их доступ должен быть ограничен. В Django суперпользователь - это наивысший уровень доступа, который Вы можете получить. Поскольку мы хотим иметь доступ ко всем частям нашего приложения, мы создадим суперпользователя. Это можно сделать с помощью команды `python manage.py createsuperuser`.

После запуска команды, Django запросит у Вас некоторую дополнительную информацию и создаст модуль `Account` с правами суперпользователя. Не бойтесь и выполните команду:

```
$ python manage.py createsuperuser
```

## Контрольная точка

Чтобы удостовериться, что все настроено правильно, давайте откроем оболочку Django в командной строке:

```
$ python manage.py shell
```

Вы должны заметить как изменилось на `>>>` приглашение командной строки. Внутри оболочки, мы можем получить доступ к только что созданной модели `Account` следующим образом:

```
>>> from authentication.models import Account
>>> a = Account.objects.latest('created_at')
```

Если не возникло никаких ошибок, то Вы должны иметь доступ к различным атрибутам Вашего объекта `Account`:

```
>>> a
>>> a.email
>>> a.username
```
