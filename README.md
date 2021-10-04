# Кодекс Питонус Доёбус

Привет! Меня зовут Всеволод, и я очень люблю доёбываться до мелочей. К счастью, я работаю в компании, где мне за это ещё и платят. Чтобы продуктивнее доёбываться, я веду список всех ошибок, которые часто встречаю на ревью у новеньких в нашей компании. Ниже не закон, а мой опыт

Самые новые ошибки вверху, чтоб легче было возвращаться к этому документу и читать его до того момента, пока не поймёшь что ниже уже читал. Поехали…


## [#16 Префиксы и постфиксы в переменных](#16)

Если переменная имеет тип datetime, то должна заканчиваться на `_at`. Если переменная имеет тип boolean, то должна начинаться на `is_`, `are_`, `was_`, `has_` и так далее. Причина в том, что если назвать переменную `deleted`, то невозможно понять: это дата удаления или флажок что оно удалено? Если переменная хранит в себе факт нужды совершить действие, например `Announcement(notify_subscribers=True)`, то переменная должна иметь префикс `do_` и называться `do_notify_subscribers`. Причина в том, что в 9 случаев из 10 со временем появляется метод `notify_subscribers` и его путают с переменной

```
❌ updated, updated, update 
⭕️ updated_at, is_updated, do_update 
```


## [#15 Слова for и to в названиях переменных](#15)

«Юзеры на удаление» переводится не как users_for_delete, а как users_to_delete. Если последнее слово глагол, то используем to, если существительное, то for

```
❌ posts_for_add, users_to_deletion
⭕️ posts_to_add, users_for_deletion
```


## [#14 Структура теста должна соответствовать AAA](#14)

Паттерн Arrange Act Assert означает что тест поделён на три части, разделённые пустыми строками — подготовка (пара строк), действие (одна строка) и асерты (сколько угодно). Таково самое лучшее деление. Часть с подготовкой может быть опущена если она выполняется на уровне фикстур

```Python
❌ 
def test_publish_post(factory):
    post = factory.post()
    post_publisher = PostPublisher(post)
    post = post_publisher()
    assert post.is_published is True
```

```Python
⭕️ 
def test_publish_post(factory):
    post = factory.post()
    post_publisher = PostPublisher(post)
    
    post = post_publisher()
    
    assert post.is_published is True
```


```Python
⭕️ 
def test_send_mail(mail):
    mail.send()
    
    assert mail.is_sent is True
```


## [#13 Бизнес-логика должна быть в сервисах](#13)

Исторически бизнес-логика хранилась в моделях, но опыт показывает что лучше её хранить в сервисах, потому что сервисы легче тестировать и поддерживать. Сервис — это небольшой класс, который наследуется от BaseService и выполняет какой-то кусочек бизнес-логики. Типичные сервисы — PostCreator, UserImporter, MembershipUpdater и так далее. Самое главное место в сервисе — метод act, перекрываем его и пишем туда код сервиса. Если нужна валидация, то расширяем метод validate. В 95% случаев сервис должен возвращать какой-то объект или объекты и обычно это тот объект или объекты, над которыми выполняется действие. Много маленьких сервисов — это хорошо

```Python
❌
class Post(DefaultModel):
    ...
    
    def publish(self):
        self.is_published = True
        self.published_at = now()
        self.save()
```

```Python
⭕️
class PostPublisher(BaseService):
    def __init__(self, post):
        self.post = post
    
    def act(self):
        self.post.is_published = True
        self.post.published_at = now()
        self.post.save()
```


## [#12 Слово when в тестах](#12)

Часто надо написать в тестах следующее: test_something_when_something. В этих сценариях лучше писать не when, а if, потому что if короче и естественнее для программиста, мы часто видим ветвление if и для нас когнитивная нагрузка от этого слова куда ниже чем от слова when

```
❌ test_membership_when_anon
⭕️ test_membership_if_anon
```


## [#11 Переменные got и result](#11)

Нельзя называть переменные именами got, result, x и так далее. Причины две. Первая: банально непонятно что внутри, это создаёт лишнюю когнитивную нагрузку на читающего. Вторая: это соблазняет не понимать собственный код. Когда разраб пишет код, он должен железно понимать его, он должен знать что внутри каждой переменной. Названия got или result соблазняют забивать на то, что внутри, а это ведёт к проблемам. Исключения у правила есть, к примеру в тестах АПИ ответ всегда называется got, потому что печатать response слишком долго, а кроме него ничего в АПИ никогда не бывает. Но если в got или result может попасть всё что угодно, то их писать нельзя, надо писать чётко что попадёт внутрь 

```
❌ got = membership_creator()
⭕️ membership = membership_creator()
```

## [#10 Впатчивание полей моделей на лету](#10)

В тестах часто бывает нужно немного отклониться от стандартной фикстурной версии модели, в результате появляется соблазн просто взять и впатчить туда нужное. От этого код становится хрупким, потому что джанговские модели устроены таким образом, что если ты впатчишь им атрибут, который на самом деле не является полем, то ошибки не случится. Если у модели есть поле author, а я опечатаюсь и в фикстуре сделаю post.athor = user, то ошибки не случится, это может породить неуловимые проблемы. Но что ещё хуже: впатчивая некий кусок данных в инстанс модели, мы тем самым относимся к модели исключительно как к данным, а не как к сущности бизнес-логики. Если при создании инстанса Post в случае указания поля author должен выполниться некий сторонний сервис, то при post.author = user он не выполнится. В результате тесты начнут тестировать искусственную функциональность, которая пропускает часть бизнес-логики. В фикстурах инстанс модели никогда не должен меняться с момента создания до момента экшена, он всегда должен доходить до экшена ровно таким, каким он был создан


```Python
❌
def test_post_of_user(post, user):
    post.author = user
    post.save()

    assert user.authored_posts.first() == post


❌
@pytest.fixture
def post_of_user(user):
    post = factory.post()
    post.author = user
    post.save()
    return post


⭕️
@pytest.fixture
def post_of_user(user):
    return factory.post(author=user)
```


## [#9 Вызов фильтраций инстанса за пределами домашней апы](#9)

Если бизнес-логика подразумевает, что публичный пост — это пост опубликованный, политкорректный и так далее, то вся эта логика должна проверяться в одном хорошо известном месте, обычно это метод менеджера. Нельзя распылять эту логику по другим приложениям, потому что тогда мы обязываем себя не забыть про все такие места, когда в понятие «публичного поста» добавится ещё какое-нибудь условие

```Python
❌ Распылять по приложениям:

# В кверисете для вывода постов на главной в апе posts
public_posts = Post.objects.filter(is_published=True, is_restricted=False)

# В кверисете для вывода постов этого автора в апе users
authored_posts = Post.objects.filter(is_published=True, is_restricted=False, author=user)

# В кверисете для вывода постов в апе sitemaps:
sitemap_posts = Post.objects.filter(is_published=True, is_restricted=False, published_at__gt=week_ago)
```

```Python
⭕️ Хранить централизованно:

# Кверисет-менеджер модели Post
class PostQuerySet(DefaultQuerySet):
    def public(self):
        return self.filter(is_published=True, is_restricted=False)


# В кверисете для вывода постов на главной в апе posts
public_posts = Post.objects.public()

# В кверисете для вывода постов этого автора в апе users
authored_posts = Post.objects.public().filter(author=user)

# В кверисете для вывода в сайтмапы:
sitemap_posts = Post.objects.public().filter(published_at__gt=week_ago)
```

## [#8 Опускание части переменной](#8)

Если сервис называется PostCreator, то когда мы инстанцируем его, мы должны назвать его post_creator, а не просто creator. Во-первых, без этого поиск post_creator по проекту пропустит это место. Во-вторых, когда человек читает код, увидев creator ему придётся гадать о том, что это за криэйтор: user_creator, post_creator, tag_creator и так далее. Не надо создавать людям лишнюю когнитивную нагрузку

```
❌ creator = PostCreator()
⭕️ post_creator = PostCreator()
```


## [#7 Глагол третьего лица в тестах](#7)

Если в названии теста используется глагол в третьем лице, то не надо ставить s на конце. Это нужно, чтобы по коду было легче найти все вхождения действия из разряда create_post. Если в поиске вбить только create_, то это выдаст кучу лишнего типа create_user, create_tag и так далее, а если вбить create_post, то поиск не найдёт тесты с названиями типа creates_post. Поэтому s не нужна

```
❌ test_user_creates_post
⭕️ test_user_create_post
```

## [#6 Типы в названиях переменных](#6)

В названии переменных не нужно указывать типы, потому что это накладывает необходимость не забыть поменять название переменной, если внутри поменяется тип данных, а это происходит повсеместно, чаще всего list превращается в dict. Другая причина — часто туда может попасть None

```
❌ user_id_list
⭕️ user_ids
```


## [#5 Несколько действий на строке](#5)

На одной строке должен быть минимум действий, иначе увеличивается когнитивная нагрузка на читающего и появляются ошибки от невнимательности к концу строки. Люди читают слева направо и фокус внимания падает по мере движения взгляда вправо

```
❌ Post.objects.published().filter(published__gte=recently_period).values_list('id', flat=True)
⭕️ Post.objects \
    .published() \
    .filter(published__gte=recently_period) \
    .values_list('id', flat=True)
```


## [#4 Магические числа](#4)

Магические числа — это числа, которые означают непонятно что. Ниже пример, где невозможно понять: семь чего? Дней, минут, часов?

```
❌ Post.objects.published_recently(7)
⭕️ Post.objects.published_recently(days=7)
```


## [#3 Нарушение noun adjunct](#3)

[Noun adjunct:](https://en.wikipedia.org/wiki/Noun_adjunct) если мы используем существительное в роли своего рода определения для другого существительного, то оно должно стоять в единственном числе. Если у нас есть массив cats и массив names, а потом мы их склеиваем, то результат должен называться cat_names, а не cats_names. Мы же например говорим backend developers, а не backends delelopers. Есть исключения типа sports companies, но в 99% случаев речь не об исключениях

```
❌ views_count
⭕️ view_count
```

```
❌ users_posts
⭕️ user_posts
```


## [#2 Слова in и from в названии переменной](#2)

Если в переменной есть in или from, то это намёк на то, что её нужно развернуть задом-наперёд. В английском почти что угодно может выполнять роль определния, поэтому более естественно писать часть после in в начале, а само in выкидывать

```
❌ posts_in_feed
⭕️ feed_posts
```

```
❌ user_from_popularity_widget
⭕️ popularity_widget_user
```

## [#1 Слово and в названии теста](#1)

Если в названии теста есть слово and, то он скорее всего тестирует больше чем одну единицу кода, его надо или разделить на два теста или слепить две единицы кода в единицу более высокого уровня

```
❌ test_first_name_and_last_name
⭕️ test_first_name, test_last_name
⭕️ test_name_fields
```
