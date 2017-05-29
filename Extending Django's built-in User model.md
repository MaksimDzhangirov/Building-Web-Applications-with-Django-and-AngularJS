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


