# Изучаем Django и AngularJS

В этом учебном пособии Вы создадите упрощенный Google+ клон под названием "Not Google Plus", используя Django и AngularJS.

Но перед тем как начать обучение и научиться создавать полноценное, современное веб приложение с помощью Django и AngularJS,
давайте уделим немного времени мотивации, побудившей написать это пособие и тому как по максимуму использовать данное учебное пособие.

## Какова цель данного учебного пособия?

Здесь в Thinkster, мы стремимся создавать высококачественные, глубокие по содержанию материалы, сохраняя при этом низкий порог вхождения.
Мы бесплатно публикуем этот материал, надеясь, что он окажется интересным и информативным для Вас.

У каждого учебного посбия, опубликованного нами, есть конкретная цель. Цель этого пособия - рассмотреть как можно совместно 
использовать Django и AngularJS и как эти технологии можно объединить, чтобы создать потрясающие воображение веб приложения. Кроме того мы 
уделяем большое внимание выработке полезных привычек, используемых при проектировании. Это включает все, начиная от учета компромиссов, 
возникающих при принятии архитектурных решений, заканчивая высоким качеством кода на протяжении всего проекта. Хотя все это может
казаться сложным, это основа, необходимая, чтобы стать полноценным разработчиком программного обеспечения.

## Для кого это учебное пособие?

Каждый автор должен ответить на этот сложный вопрос. Наша цель сделать это учебное пособие полезным как для новичков, так и для опытных разработчиков.

Для тех из Вас, кто только начинает свою карьеру разработчика программного обеспечения, мы постарались сделать наши объяснения как можно более подробными и логичными, сохраняя при этом плавное изложение материала; по возможности мы старались не делать резких скачков, опуская материал, интуитивно понятный опытным разработчикам, но вызывающий сложности у новичков.

Для тех из Вас, кто уже хорошо знаком с описанными технологиями, и возможно просто хочет узнать что-то новое о Django и AngularJS, мы понимаем, что Вам не нужно объяснять основы. Одной из наших целей при написании данного учебного пособия разбить его на блоки, чтобы информацию было проще структурировать. Это позволит, используя Ваши текущие знания, ускорить процесс чтения, быстро определяя места в тексте, с незнакомыми Вам понятиями. Таким образом, Вы можете быстрее разобраться в них и двигаться дальше.

Мы хотим сделать это пособие доступным для всех, кто заинтересован и готов потратить время на изучение понимания представленных понятий.

## Небольшое замечание, касающееся форматирования текста

Во всем учебном пособии, мы стараемся придерживаться одного и то же стиля форматирования. В этом разделе описывается как будет выглядеть форматирование и что оно означает.

* Каждый раз при предоставлении нового фрагмента кода, мы сначала показываем весь код целиком, а затем подробно рассматриваем его построчно в тех местах, где встречаются новые понятия.
* Названия переменных и файлов, которые встречаются в предложении, выделяются следующим образом: `thinkster_django_angular / settings.py.`
* Большие фрагменты кода располагаются отдельно:
```python
def is_this_google_plus():
    return False
```
* Команды, используемые в терминале, также выносятся на отдельную строку, начанающуюся с $:
``` 
$ python manage.py runserver 
```
* Если нет явных указаний, то предполагается, что все команды в терминале выполняются из корневого каталога Вашего проекта.

## Небольшая информация о стандартах кодирования

По возможности мы стараемся придерживаться стандартов кодирования, созданных сообществами Django и AngularJS.

Для Django мы строго следуем [PEP8](http://legacy.python.org/dev/peps/pep-0008/) и стараемся придерживаться стиля кодирования Django.

Для AngularJS, мы использовали [руководство по стандарту кодирования AngularJS](https://github.com/johnpapa/angularjs-styleguide) от John Papa. Также, где это имело смысл, мы придерживались руководства по [стилю кодирования JavaScript от Google](https://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml).

## Будем признательны за любую обратную связь

Как бы пафосно это не звучало, единственная причина создания этого пособия - это Вы, наши читатели. Поскольку мы считаем, что Ваши успехи - это наши успехи тоже, мы просим Вас связаться с нами и поделиться своими мыслями, впечатлениями об этом пособии. Вы можете сделать это через Twitter [@jamesbrwr](http://twitter.com/jamesbrwr) или [@GoThinkster](http://twitter.com/gothinkster), а также по электронной почте support@thinkster.io.

Мы привествуем критику и благодарны за похвалу, если Вы действительно считаете, что мы её заслуживаем. Нам интересно было бы узнать, что Вам понравилось, что не понравилось, о чем бы Вы ещё хотели бы узнать, а также о обо всем другом, о чем бы Вы хотели сообщить.

Если Вы слишком заняты, чтобы связаться с нами, ничего страшного. Мы знаем, что обучение занимает много времени. С другой стороны, если Вы хотите помочь нам создать что-то действиетльно потрясающее, мы ждем Ваших писем.

## Заключительные слова, прежде чем начать изложение материала

Наш опыт показывает, что максимальную пользу от наших пособий, получают те разработчики, которые закрепляют на практике навыки, полученные в ходе обучения.

Мы **настоятельно** рекомендуем Вам вводить код _вручную_. Когда Вы копируете и вставляете код, Вы не взаимодействуете с ним, а это взаимодействие как раз и делает Вас лучшим разработчиком по сравнению с другими.

Кроме того, не бойтесь изменять код; добавляйте или удаляйте что-то, создавайте недостающий функционал. Если Вы столкнулись с ошибкой, изучите и выясните, что её вызывает. Это работа, с которой Вы как разработчики, будете сталкиваться постоянно, поэтому умение справляться с ошибками - одно из важнейших результатов обучения.

Итак давайте начнем создавать программное обеспечение.
