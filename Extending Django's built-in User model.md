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
  tagline=models.CharField(max_length=140, blank=True)
  
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


