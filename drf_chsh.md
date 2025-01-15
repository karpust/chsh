# drf-my-cheat-sheet
Custom cheat-sheet for DRF tools and features

мой Cheat Sheet по Django REST Framework. Здесь будут мои записи для скорейшего освоения drf.

## Содержание
1. [Сериализация (Serialization)](#сериализация-serialization)
   - [1.1. Serializer](#11-serializer)
     - [Создание сериализатора](#создание-сериализатора)
     - [Сериализация объектов](#сериализация-объектов)
     - [Десериализация объектов](#десериализация-объектов)
     - [Сохранение экземпляров модели](#сохранение-экземпляров-модели)
     - [Валидация](#валидация)
     - [Доступ к instance and initial data](#доступ-к-instance-and-initial-data)
     - [Частичное обновление](#частичное-обновление)
     - [Вложенные сложные объекты](#вложенные-сложные-объекты)
     - [Записываемые вложенные представления](#записываемые-вложенные-представления)
     - [Сериализация нескольких объектов](#сериализация-нескольких-объектов)
     - [Включение дополнительного контекста](#включение-дополнительного-контекста)
   - [1.2. ModelSerializer](#12-modelserializer)
     - [Выбор полей](#выбор-полей)
     - [Указание вложенной сериализации](#указание-вложенной-сериализации)
     - [Явный выбор полей](#явный-выбор-полей)
     - [Указание полей для чтения](#указание-полей-для-чтения)
     - [Дополнительные аргументы и изменение полей модели](#дополнительные-аргументы-и-изменение-полей-модели)
     - [Настройка сопоставления полей](#настройка-сопоставления-полей)
     - [Классы и аргументы полей](#классы-и-аргументы-полей)
   - [1.3. HyperlinkedModelSerializer](#13-hyperlinkedmodelserializer)
     - [Абсолютные и относительные ссылки](#абсолютные-и-относительные-ссылки)
     - [Как определяются представления для гиперссылок](#как-определяются-представления-для-гиперссылок)
   - [1.4. ListSerializer](#14-listserializer)
     - [Настройка поведения ListSerializer](#настройка-поведения-listserializer)
     - [Настройка инициализации ListSerializer при неявном объявлении](#настройка-инициализации-listserializer-при-неявном-объявлении)
   - [1.5. BaseSerializer](#15-baseserializer)
     - [Read-only сериализатор на основе BaseSerializer](#read-only-сериализатор-на-основе-baseserializer)
     - [Read-write BaseSerializer](#read-write-baseserializer)
   - [1.6. Advanced serializer usage](#16-advanced-serializer-usage)
     - [Переопределение поведения сериализации и десериализации](#переопределение-поведения-сериализации-и-десериализации)
     - [Наследование сериализатора](#наследование-сериализатора)
     - [Динамическое изменение полей](#динамическое-изменение-полей)
   - [1.7. Отношения сериализатора](#17-отношения-сериализатора)
     - [Проверка отношений](#проверка-отношений)
     - [API Reference](#api-reference)
       - [StringRelatedField](#stringrelatedfield)
       - [PrimaryKeyRelatedField](#primarykeyrelatedfield)
       - [HyperlinkedRelatedField](#hyperlinkedrelatedfield)
       - [SlugRelatedField](#slugrelatedfield)
       - [HyperlinkedIdentityField](#hyperlinkedidentityfield)
     - [Примечания](#римечания)
       - [queryset](#queryset)
       - [Настройка отображения HTML](#настройка-отображения-html)
       - [Отсечение полей](#отсечение-полей)
       - [Обратные отношения](#обратные-отношения)
       - [Общие отношения](#общие-отношения)
       - [ManyToManyFields с промежуточной моделью](#manytomanyfields-с-промежуточной-моделью)
2. [Routers](#routers)
3. [Authentication](#authentication)
4. [Permissions](permissions)
5. [Validators](#validators)
   - [Виды валидаторов DRF](#виды-валидаторов-drf)
   - [Ограничения валидаторов в Django REST Framework](#ограничения-валидаторов-в-django-rest-framework)
   - [Кастомные валидаторы](#кастомные-валидаторы)







2. [Виды (Views)](#виды-views)
3. [Аутентификация (Authentication)](#аутентификация-authentication)
4. [Разрешения (Permissions)](#разрешения-permissions)

---

## Сериализация (Serialization)
преобразуют сложные типы данных(наборы запросов, экз моделей) в типы данных питона(словарь) и обратно.
которые потом можно преобразовать в .json и отдать через api.
в них передают модель и определяют какие ее поля будут преобразовываться.

### 1.1. Serializer
Базовый класс для создания сериализаторов:
#### Создание сериализатора

```python

from rest_framework import serializers
from datetime import datetime

# имеется экземпляр модели:
class Comment:
    def __init__(self, email, content, created=None):
        self.email = email
        self.content = content
        self.created = created or datetime.now()

comment = Comment(email='leila@example.com', content='foo bar')


# объявление сериализатора для модели:
class CommentSerializer(serializers.Serializer):
    email = serializers.EmailField()
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
```   

#### Сериализация объектов

```python   
# передаем в сериализатор экземпляр модели, получаем словарь:
serializer = CommentSerializer(comment)
serializer.data
# {'email': 'leila@example.com', 'content': 'foo bar', 'created': '2016-01-27T15:17:10.375877'}


# из словаря получаем .json:
from rest_framework.renderers import JSONRenderer

json = JSONRenderer().render(serializer.data)
json
# b'{"email":"leila@example.com","content":"foo bar","created":"2016-01-27T15:17:10.375877"}'
```

#### Десериализация объектов

```python
import io
from rest_framework.parsers import JSONParser

# преобразуем поток в типы данных python(json=>dict):
stream = io.BytesIO(json)  # BytesIO
data = JSONParser().parse(stream)  # dict

# словарь отдаем сериалайзеру(с инстансом - обновление, без инстанса - создание нового объекта)
serializer = CommentSerializer(data=data)
serializer.is_valid()  # if True => validated_data становится доступен
serializer.validated_data  # dict, проверенные данные которые сохраняет в себе сериалайзер 
# чтобы потом их использовать при создании экз модели.
# {'content': 'foo bar', 'email': 'leila@example.com', 'created': datetime.datetime(2012, 08, 22, 16, 20, 09, 822243)}
comment = serializer.save() # сохраняем джанго модель, см ниже
```
#### Сохранение экземпляров модели
чтобы использовать данные которые получаем (десериализация) нужно иметь возможность 
создавать или обновлять экземпляр модели, для этого методы create, update)
```python
class CommentSerializer(serializers.Serializer):
    email = serializers.EmailField()
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()

    def create(self, validated_data):
        return Comment(**validated_data)

    def update(self, instance, validated_data):
        instance.email = validated_data.get('email', instance.email)
        instance.content = validated_data.get('content', instance.content)
        instance.created = validated_data.get('created', instance.created)
        return instance

# для моделей джанги сохраняем данные в бд:
def create(self, validated_data):
        return Comment.objects.create(**validated_data)  
        # save data in db and return model's instance

    def update(self, instance, validated_data):
        instance.email = validated_data.get('email', instance.email)
        instance.content = validated_data.get('content', instance.content)
        instance.created = validated_data.get('created', instance.created)
        instance.save()  # save data in db and return model's instance
        return instance

# .save() will create a new instance.
serializer = CommentSerializer(data=data)

# .save() will update the existing `comment` instance.
serializer = CommentSerializer(comment, data=data)

# в save() можно передать доп данные, они передадутся в validated_data:
serializer.save(owner=request.user)
```
[к содержанию](#содержание)
#### Валидация
Можно использовать простые валидаторы(в классе сериализатора) или reusable валидаторы(определены вне сериализатора).
Все они могут применяться на уровне поля или объекта.
 
```python
# в случае ошибки вернет словарь где ключ-поле, значение-список с ошибками для этого поля:
serializer = CommentSerializer(data={'email': 'foobar', 'content': 'baz'})
serializer.is_valid()  # False
serializer.errors
# {'email': ['Enter a valid e-mail address.'], 'created': ['This field is required.']}

from rest_framework import serializers

class BlogPostSerializer(serializers.Serializer):
    title = serializers.CharField(max_length=100)
    content = serializers.CharField()

    def validate_title(self, value):
        """
        Check that the blog post is about Django.
        """
        if 'django' not in value.lower():
            raise serializers.ValidationError("Blog post is not about Django")
        return value
```

#### Доступ к instance and initial data
```python
# .instance содержит объект (или queryset), который передан в сериализатор:

class UserSerializer(serializers.Serializer):
    username = serializers.CharField()
    email = serializers.EmailField()

user = {'username': 'john_doe', 'email': 'john@example.com'}

# Создаём экземпляр сериализатора с объектом
serializer = UserSerializer(instance=user)

# Доступ к атрибуту .instance
print(serializer.instance)
# Вывод: {'username': 'john_doe', 'email': 'john@example.com'}

# Если instance не передан
empty_serializer = UserSerializer()
print(empty_serializer.instance)
# Вывод: None

# .initial_data содержит необработанные данные, переданные в сериализатор
# через аргумент data. Эти данные ещё не прошли валидацию:
input_data = {'username': 'john_doe', 'email': 'john@example.com'}
serializer = UserSerializer(data=input_data)

# Доступ к атрибуту .initial_data
print(serializer.initial_data)
# {'username': 'john_doe', 'email': 'john@example.com'}

# Если данные не переданы
empty_serializer = UserSerializer()
print(hasattr(empty_serializer, 'initial_data'))  # Проверка существования атрибута
# False
```
#### Частичное обновление
```python
# Update `comment` with partial data - must include partial=True:
serializer = CommentSerializer(comment, data={'content': 'foo bar'}, partial=True)
```

#### Вложенные сложные объекты
поле сериализатора может быть не простым объектом, а тоже сериализатором, None или списком:  
```python
class UserSerializer(serializers.Serializer):
    email = serializers.EmailField()
    username = serializers.CharField(max_length=100)

class CommentSerializer(serializers.Serializer):
    user = UserSerializer()  # если поле есть сериалайзер
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
    
class CommentSerializer(serializers.Serializer):
    user = UserSerializer(required=False)  # если поле None
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()

class CommentSerializer(serializers.Serializer):
    user = UserSerializer(required=False)
    edits = EditItemSerializer(many=True)  # если поле это список
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
```
#### Записываемые вложенные представления
Если возникают ошибки во вложенных полях, то они отображаются с исходной вложенностью;
так же будет и с полем .validated_data:
```python
serializer = CommentSerializer(data={'user': {'email': 'foobar', 'username': 'doe'}, 'content': 'baz'})
serializer.is_valid()
# False
serializer.errors
# {'user': {'email': ['Enter a valid e-mail address.']}, 'created': ['This field is required.']}
```
Если используются вложенные объекты, то методы .create() and update() нужно переопределить:
```python
# create:
class UserSerializer(serializers.ModelSerializer):
    profile = ProfileSerializer()

    class Meta:
        model = User
        fields = ['username', 'email', 'profile']

    def create(self, validated_data):
        profile_data = validated_data.pop('profile')
        user = User.objects.create(**validated_data)
        Profile.objects.create(user=user, **profile_data)
        return user

# update:
def update(self, instance, validated_data):
        profile_data = validated_data.pop('profile')
        # Unless the application properly enforces that this field is
        # always set, the following could raise a `DoesNotExist`, which
        # would need to be handled.
        profile = instance.profile

        instance.username = validated_data.get('username', instance.username)
        instance.email = validated_data.get('email', instance.email)
        instance.save()

        profile.is_premium_member = profile_data.get(
            'is_premium_member',
            profile.is_premium_member
        )
        profile.has_support_contract = profile_data.get(
            'has_support_contract',
            profile.has_support_contract
         )
        profile.save()

        return instance
```
[к содержанию](#содержание)

#### Сериализация нескольких объектов
```python
queryset = Book.objects.all()
serializer = BookSerializer(queryset, many=True)
serializer.data
# [
#     {'id': 0, 'title': 'The electric kool-aid acid test', 'author': 'Tom Wolfe'},
#     {'id': 1, 'title': 'If this is a man', 'author': 'Primo Levi'},
#     {'id': 2, 'title': 'The wind-up bird chronicle', 'author': 'Haruki Murakami'}
# ]
```
Десериализация нескольких объектов поддерживается по умолчанию на создание, но не на обновление.

#### Включение дополнительного контекста
В дополнение к сериализуемому объекту бывает нужно передать дополнительный 
контекст (контекстный словарь), например если поле сериализатора включает гиперссылку:
```python
serializer = AccountSerializer(account, context={'request': request})
serializer.data
# {'owner': 'denvercoder9', 'details': 'http://example.com/accounts/6/details'}
```

### 1.2. ModelSerializer
Если есть модель джанго, то этот класс автоматически создаст сериализатор с полями из этой модели, встроенной валидацией, и встроенными методами create, update:
```python
# объявление сериализатора:
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = ['id', 'account_name', 'users', 'created']
```
Все отношения модели(внешние ключи) передадутся в поле типа PrimaryKeyRelatedField.
Обратные отношения требуют явного включения.
```python
# посмотреть созданный сериализатор и его поля:
# python manage.py shell:
>>> from myapp.serializers import AccountSerializer
>>> serializer = AccountSerializer()
>>> print(repr(serializer))
AccountSerializer():
    id = IntegerField(label='ID', read_only=True)
    name = CharField(allow_blank=True, max_length=100, required=False)
    owner = PrimaryKeyRelatedField(queryset=User.objects.all())
```
#### Выбор полей
Можно выбрать какие поля модели передавать в сериализатор:
```python
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = ['id', 'account_name', 'users', 'created']
        # fields = '__all__'  # или все поля
```
```python
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        exclude = ['users']  # исключить поля
```

#### Указание вложенной сериализации
`depth` указывает уровень вложенности связанных объектов, которые будут сериализованы.
По умолчанию ModelSerializer использует первичные ключи для указания отношений, но
если поле связано с другой моделью или является тоже сериализатором, тогда в это поле будет включен не только id, а все поля той модели(сериализатора).
```python
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = ['id', 'account_name', 'users', 'created']
        depth = 1
```
```json
{
    "id": 1,
    "account_name": "My Account",
    "users": [
        {
            "id": 10,
            "username": "johndoe",
            "email": "johndoe@example.com"
        }
    ],
    "created": "2024-12-28T12:34:56Z"
}

```
Однако, большая глубина замедляет процесс сериализации.
#### Явный выбор полей
Если нужно добавить поля которых нет в модели или переопределить существующие
```python
class AccountSerializer(serializers.ModelSerializer):
    url = serializers.CharField(source='get_absolute_url', read_only=True)
    groups = serializers.PrimaryKeyRelatedField(many=True)

    class Meta:
        model = Account
        fields = ['url', 'groups']
```
#### Указание полей для чтения
Идивидуально для поля при его объявлении указать `read_only=True`.
Или передать в списке несколько полей через `read_only_fields`
```python
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = ['id', 'account_name', 'users', 'created']
        read_only_fields = ['account_name']
```
Полями для чтения по умолчанию будут поля типа `AutoField` и поля со свойством `editable=False`.
Если поле, которое должно быть только для чтения, является частью ограничение `unique_together`, то ему нужно установить `read_only=True` и задать значение по-умолчанию: 

```python
# В джанго модели поля, которые должны быть уникальны в своей комбинации, перечисляются в `unique_together`
class MyModel(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    identifier = models.CharField(max_length=100)

    class Meta:
        unique_together = ('user', 'identifier')
```
Все поля находящиеся в `unique_together` будут участвовать в валидации, тк drf должен проверить, что выполняются условия этого ограничения. 
Если просто запретить редактирование через `read_only=True`, то при создании нового объекта валидация провалится, тк значение поля окажется незаданным, поэтому полю нужно определить значение по умолчанию.
```python
class MySerializer(serializers.ModelSerializer):
    user = serializers.PrimaryKeyRelatedField(
        read_only=True,
        default=serializers.CurrentUserDefault()
        # в поле юзер передастся текущий юзер
    )

    class Meta:
        model = MyModel
        fields = ['user', 'identifier']
```
[к содержанию](#содержание)

#### Дополнительные аргументы и изменение полей модели
Чтобы изменить поля модели переданные в сериализатор, нужно их передать в `extra_kwargs`
```python
class CreateUserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['email', 'username', 'password']
        extra_kwargs = {'password': {'write_only': True}}

    def create(self, validated_data):
        user = User(
            email=validated_data['email'],
            username=validated_data['username']
        )
        user.set_password(validated_data['password'])
        user.save()
        return user
```
`extra_kwargs` нужны для изменения существующих полей модели, поэтому
изменения применятся при неявном объявлении поля(передано из модели), и не применятся при явном объявлении(в сериализаторе)
```python
class MyModelSerializer(serializers.ModelSerializer):
    name = serializers.CharField(read_only=True)  # Поле объявлено явно.

    class Meta:
        model = MyModel
        fields = ['name', 'description']
        extra_kwargs = {
            'name': {'write_only': True},  # Настройки будут проигнорированы.
        }
```
#### Настройка сопоставления полей
`serializer_field_mapping` - это атрибут класса ModelSerializer, который представляет собой словарь, где ключи — это типы полей модели (models.Field), а значения — соответствующие классы полей сериализатора (serializers.Field). Используется для быстрого сопоставления и создания кастомных полей.
```python
from django.db import models
from rest_framework import serializers

serializer_field_mapping = {
    models.AutoField: serializers.IntegerField,
    models.CharField: serializers.CharField,
    models.TextField: serializers.CharField,
    models.IntegerField: serializers.IntegerField,
    models.FloatField: serializers.FloatField,
    models.BooleanField: serializers.BooleanField,
    models.DateField: serializers.DateField,
    models.DateTimeField: serializers.DateTimeField,
    models.ForeignKey: serializers.PrimaryKeyRelatedField,
    # и другие поля
}
```
создание кастомного поля:
```python
from rest_framework import serializers

class CustomCharField(serializers.CharField):
    def to_representation(self, value):
        return value.upper()  # Все значения переводятся в верхний регистр.
```
переопределение сопоставления:
```python
from rest_framework import serializers
from django.db import models

class MyModelSerializer(serializers.ModelSerializer):
    # Переопределяем маппинг, не изменяя оригинальный
    serializer_field_mapping = serializers.ModelSerializer.serializer_field_mapping.copy()
    serializer_field_mapping[models.CharField] = CustomCharField

    class Meta:
        model = MyModel
        fields = '__all__'
```
`serializer_related_field` - определяет какой тип поля сериализатора будет использован для отношений(ForeignKey, ManyToManyField, OneToOneField) по умолчанию.
для `ModelSerializer` используется `serializers.PrimaryKeyRelatedField` и означает, что связанные объекты будут представлены их первичным ключом.

For `HyperlinkedModelSerializer` this defaults to `serializers.HyperlinkedRelatedField` и означает что связанное поле будет представлено как гиперссылка на URL-адрес, связанный с этим объектом.

`serializer_url_field` - атрибут класса который используется для url-полей, его значение по умолчанию serializers.HyperlinkedIdentityField

`serializer_choice_field` - `Defaults to serializers.ChoiceField`

#### Классы и аргументы полей
В API Django REST Framework методы для генерации полей сериализатора автоматически определяют класс поля и аргументы для каждого поля модели. Они возвращают кортеж: (field_class, field_kwargs).

- `build_standard_field`(self, field_name, model_field)
Генерирует поле сериализатора для стандартного поля модели.
По умолчанию выбирает класс поля на основе атрибута `serializer_field_mapping`.
- `build_relational_field`(self, field_name, relation_info)
Создаёт поле для отношений (ForeignKey, ManyToManyField и т.д.).
Использует `serializer_related_field` для выбора класса поля.
Аргумент `relation_info` — именованный кортеж с информацией:
`model_field`: поле модели.
related_model: связанная модель.
to_many: булево значение (множественное отношение или нет).
has_through_model: булево значение (есть ли промежуточная модель, например, в ManyToManyField с through).
- `build_nested_field`(self, field_name, relation_info, nested_depth)
Генерирует поле для вложенного сериализатора, если установлена опция depth.
Динамически создаёт вложенный сериализатор (на основе `ModelSerializer` или `HyperlinkedModelSerializer`).
Аргумент `nested_depth`: текущая глубина вложенности (`depth` - 1).
Использует `relation_info`, аналогично `build_relational_field`.
- `build_property_field`(self, field_name, model_class)
Генерирует поле для свойства или метода модели без аргументов.
По умолчанию возвращает поле `ReadOnlyField`.
- `build_url_field`(self, field_name, model_class)
Создаёт поле для ссылки на сам объект (поле url).
По умолчанию возвращает `HyperlinkedIdentityField`.
- `build_unknown_field`(self, field_name, model_class)
Вызывается, если поле не связано с полем модели или свойством.
По умолчанию вызывает ошибку.
Можно переопределить, чтобы обрабатывать такие случаи.

[к содержанию](#содержание)

### 1.3. HyperlinkedModelSerializer
`HyperlinkedModelSerializer` похож на `ModelSerializer`, но для отношений использует гиперссылки, а не первичные ключи.
вместо поля с первичным ключом имеет поле с url по умолчанию:
поле url имеет тип `HyperlinkedIdentityField`, а отношения реализуются полем `HyperlinkedRelatedField`

```python
class AccountSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Account
        fields = ['url', 'id', 'account_name', 'users', 'created']
        # так первичный ключ можно задать явно (id)
```
`HyperlinkedIdentityField` получает ссылку на сам объект(ссылку на конкретную книгу):
```python
class BookSerializer(serializers.HyperlinkedModelSerializer):
    url = serializers.HyperlinkedIdentityField(view_name='book-detail')
    # view_name='book-detail' указывает на имя view, которое используется 
    # для отображения данных конкретной книги.
    
    class Meta:
        model = Book
        fields = ['url', 'title', 'author']

```
```json
{
    "url": "http://example.com/books/1/",
    "title": "The Great Gatsby",
    "author": "F. Scott Fitzgerald"
}
```
`HyperlinkedRelatedField` получает ссылку на связанный объект(ссылка на автора, связанного с книгой):
```python
class BookSerializer(serializers.HyperlinkedModelSerializer):
    author = serializers.HyperlinkedRelatedField(queryset=Author.objects.all(), view_name='author-detail')

    class Meta:
        model = Book
        fields = ['url', 'title', 'author']

```
```json
{
    "url": "http://example.com/books/1/",
    "title": "The Great Gatsby",
    "author": "http://example.com/authors/1/"
}
```

#### Абсолютные и относительные ссылки
При создании экземпляра класса `HyperlinkedModelSerializer` нужно передать контекстный словарь с запросом, чтобы гиперссылка была полным адресом url - содержала в себе имя хоста, а не была относительной ссылкой:
```python
serializer = AccountSerializer(queryset, context={'request': request})
# http://api.example.com/accounts/1/
# rather than: /accounts/1/
```
Если нужно использовать относительную ссылку, в контекстный словарь нужно передать `{'request': None}`

#### Как определяются представления для гиперссылок
Нужно определить какие представления использовать для гиперссылок на экземпляры модели. 
По умолчанию ожидается, что гиперссылка будет соответствовать стилю `{имя_модели}-detail` и искать экземпляр по первичному ключу.
Такое поведение можно изменить, переопределив параметры `view_name` и `lookup_field`:

объявив их неявно (используя `extra_kwargs`) :
```python
class AccountSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Account
        fields = ['account_url', 'account_name', 'users', 'created']
        extra_kwargs = {
            'url': {'view_name': 'accounts', 'lookup_field': 'account_name'},
            'users': {'lookup_field': 'username'}
        }
        # поле url будет генерировать гиперссылку на основе view_name='accounts' 
        # и значения поля account_name вместо дефолтного pk
        # http://api.example.com/accounts/my-account-name/
        # вместо: 
        # http://api.example.com/accounts/1/
```

или объявив явно:
```python
class AccountSerializer(serializers.HyperlinkedModelSerializer):
    url = serializers.HyperlinkedIdentityField(
        view_name='accounts',
        lookup_field='slug'
    )
    users = serializers.HyperlinkedRelatedField(
        view_name='user-detail',
        lookup_field='username',
        many=True,
        read_only=True
    )

    class Meta:
        model = Account
        fields = ['url', 'account_name', 'users', 'created']
```
`lookup_field` - атрибут, который обозначает какое поле модели использовать для поиска объекта, по-умолчанию pk.
`view_name` — это атрибут, с именем представления, которое будет использовано для формирования URL гиперссылки.
Чтобы проконтролировать правильную настройку имен представлений (view names) и полей для поиска (lookup fields) для гиперссылки, нужно использовать отладку с repr() для экземпляра `HyperlinkedModelSerializer`.

### 1.4. ListSerializer
`ListSerializer` позволяет сериализовать и валидировать несколько объектов одновременно. Обычно его не вызывают явно, но если установить `many=True` при создании экземпляра сериализатора, то будет вызван класс `ListSerializer` и создан его экземпляр.

еще аргументы `ListSerializer`:

`allow_empty` - разрешает пустой список, True by default

`max_length` - ограничивает максимальную длину списка, None by default

`min_length` - ограничивает минимальную длину списка, None by default

#### Настройка поведения `ListSerializer`
При множественном создании вызывается `create()` for each item in the list.
```python
# встроенный create():
def create(self, validated_data):
    return [self.child.create(attrs) for attrs in validated_data]
    # child.create() вызывает create() дочернего сериализатора.
    # для каждого элемента отдельный запрос в базу данных.
```
можно переопределить встроенный create():
```python
class BookListSerializer(serializers.ListSerializer):
    def create(self, validated_data):
        books = [Book(**item) for item in validated_data]
        return Book.objects.bulk_create(books)
        # массовое создание объектов с методом bulk_create():
        # используется всего один запрос к бд

class BookSerializer(serializers.Serializer):
    ...
    class Meta:
        list_serializer_class = BookListSerializer
```
[к содержанию](#содержание)

Множественное обновление не поддерживается классом `ListSerializer` by default.
Тк dfr не может сам решить что нужно создать, что обновить, что удалить. 
Это нужно указывать явно.

Для этого нужно явно объявить поле `id`, тк при неявном создании оно `read_only` и удаляется при обновлении. 

При явном объявлении поле `id` доступно в методе `update()`
```python
class BookListSerializer(serializers.ListSerializer):
    def update(self, instance, validated_data):
        # Maps for id->instance and id->data item.
        book_mapping = {book.id: book for book in instance}  # объекты из бд
        data_mapping = {item['id']: item for item in validated_data}  # входящие данные

        # Perform creations and updates.
        ret = []
        for book_id, data in data_mapping.items():
            book = book_mapping.get(book_id, None)
            if book is None:
                ret.append(self.child.create(data))
                # если объекта нет в бд, он создается
            else:
                ret.append(self.child.update(book, data))
                # если есть, то обновляется

        # Perform deletions.
        for book_id, book in book_mapping.items():
            if book_id not in data_mapping:
            # если объекта базы данных нет во входящих данных, удаляем его из бд
                book.delete()

        return ret

class BookSerializer(serializers.Serializer):
    # We need to identify elements in the list using their primary key,
    # so use a writable field here, rather than the default which would be read-only.
    id = serializers.IntegerField()
    ...

    class Meta:
        list_serializer_class = BookListSerializer
```

#### Настройка инициализации `ListSerializer` при неявном объявлении
нужно понять какие аргументы будут переданы в __init__() для существующего(дочернего) сериализатора и для будущего(родительского) `ListSerializer`.
По умолчанию передаются все аргументы кроме валидаторов и пользовательских именованных аргументов(они только для дочернего).

Можно это изменить и передать в родительский то, что нужно:
```python
@classmethod
    def many_init(cls, *args, **kwargs):
        # Instantiate the child serializer.
        kwargs['child'] = cls()
        # Instantiate the parent list serializer.
        return CustomListSerializer(*args, **kwargs)
```

### 1.5. BaseSerializer
`BaseSerializer` — базовый класс, который не предоставляет встроенной логики для работы с полями, валидацией или моделями. Требует явной реализации методов `to_representation` и `to_internal_value` для сериализации и десериализации данных. Имеет более низкий уровень абстракции чем `Serializer`. Используется для более кастомизированных случаев, когда остальные не подходят.

Методы которые нужно реализовать, тк по умолчанию вернут NotImplementedError:

`to_representation()` - реализовать метод, для сериализации (read only)
(преобразование объекта сериализации в читаемый формат).

`to_internal_value()` - реализовать метод, для десериализации (write)

`create()`, `update()` - реализовать методы, чтобы сохранять экземпляры модели.

#### Read-only сериализатор на основе BaseSerializer
для создания read-only сериализатора нужно реализовать метод `to_representation()`. 
Он отвечает за преобразование объекта модели в словарь данных, который будет возвращен клиенту.
```python
class HighScore(models.Model):
    # есть модель
    created = models.DateTimeField(auto_now_add=True)
    player_name = models.CharField(max_length=10)
    score = models.IntegerField()
```
```python
class HighScoreSerializer(serializers.BaseSerializer):
    def to_representation(self, instance):
        # instance это экземпляр модели
        # преобразует объект модели в словарь и вернет его
        return {
            'score': instance.score,
            'player_name': instance.player_name
        }
```
представление для одного объекта модели:
```python
@api_view(['GET'])
def high_score(request, pk):
    instance = HighScore.objects.get(pk=pk)
    serializer = HighScoreSerializer(instance)
    return Response(serializer.data)
```

представление для всех объектов модели:
```python
@api_view(['GET'])
def all_high_scores(request):
    queryset = HighScore.objects.order_by('-score')
    serializer = HighScoreSerializer(queryset, many=True)
    return Response(serializer.data)
```
[к содержанию](#содержание)

#### Read-write BaseSerializer
Для создания read-write сериализатора нужно реализовать метод `to_internal_value()`.
Он преобразовывает данные, полученные из запроса, во внутренние данные, для сохранения в бд, создания экземпляров модели.
Метод `to_internal_value()` делает доступной валидацию с методами `is_valid()`, `validated_data`, `errors`. 

```python
class HighScoreSerializer(serializers.BaseSerializer):
    def to_internal_value(self, data):
        score = data.get('score')
        player_name = data.get('player_name')

        # Perform the data validation.
        if not score:
            raise serializers.ValidationError({
                'score': 'This field is required.'
            })
        if not player_name:
            raise serializers.ValidationError({
                'player_name': 'This field is required.'
            })
        if len(player_name) > 10:
            raise serializers.ValidationError({
                'player_name': 'May not be more than 10 characters.'
            })

        # Return the validated values. This will be available as
        # the `.validated_data` property.
        return {
            'score': int(score),
            'player_name': player_name
        }

    def to_representation(self, instance):
        return {
            'score': instance.score,
            'player_name': instance.player_name
        }

    def create(self, validated_data):
        return HighScore.objects.create(**validated_data)
```

### 1.6. Advanced serializer usage

#### Переопределение поведения сериализации и десериализации
в классе Serializer определены методы `to_representation`(отвечает за сериализацию) и `to_internal_value`(отвечает за десериализацию), которые можно переопределить.

`to_representation(self, instance)` - принимает объект модели, возвращает питоновский тип данных(зависит от настроек рендеринга).

```python
def to_representation(self, instance):
    """Convert `username` to lowercase."""
    ret = super().to_representation(instance)
    ret['username'] = ret['username'].lower()
    return ret
```

`to_internal_value(self, data)` - принимает невалидированные данные из запроса, возвращает проверенные данные, которые доступны в `serializer.validated_data` и передаются в методы `create()` или `update()`, если вызван `save()`.

`data`является значением request.data, его тип данных зависит от настроек парсера.

#### Наследование сериализатора

Сериализаторы можно расширить и переиспользовать применяя наследование. Для этого у родительского класса нужно объявить общие поля и методы, которые понадобятся детям.
Мета-класс не наследуется неявно, чтобы он наследовался нужно сделать это явно(не рекомендуется):
```python
class AccountSerializer(MyBaseSerializer):
    class Meta(MyBaseSerializer.Meta):
    # наследоваться в Meta не рекомендуется
        model = Account
```
```python
class BookSerializer(serializers.ModelSerializer):
    class Meta:
    # лучше не наследоваться, а объявлять явно параметры Meta для каждого сериализатора:
        model = Book
        fields = ['title', 'author', 'published_date', 'isbn']
        read_only_fields = ['author']
```

Можно явно удалить поле, указав значение None, если оно было явно определено в родительском классе.
Если же поле было получено из модели, оно не удалится - нужно использовать [Выбор полей](#выбор-полей).
```python
class MyBaseSerializer(ModelSerializer):
    my_field = serializers.CharField()

class MySerializer(MyBaseSerializer):
    my_field = None
```

#### Динамическое изменение полей
Получение доступа к полям объекта сериализатора, используя аттрибут `.fields` - он позволяет менять поля во время выполнения.

Создадим сериализатор, позволяющий задать поля, которые будут созданы при его инициализации:
```python
class DynamicFieldsModelSerializer(serializers.ModelSerializer):
    """
    A ModelSerializer that takes an additional `fields` argument that
    controls which fields should be displayed.
    """

    def __init__(self, *args, **kwargs):
        fields = kwargs.pop('fields', None)
        # сохраняем поля полученные из UserSerializer(user, fields=('id', 'email'))
        # в fields и удаляем их из kwargs,
        # чтобы не передавать в родительский класс (ModelSerializer),
        # который не принимает такие аргументы.

        super().__init__(*args, **kwargs)
        # для инициалилзации используем конструктор родителя (ModelSerializer)
        # с полями определенными в Meta UserSerializer

        if fields is not None:
            # Drop any fields that are not specified in the `fields` argument.
            allowed = set(fields)  # поля из UserSerializer(user, fields=('id', 'email'))
            existing = set(self.fields) # поля из Meta UserSerializer
            for field_name in existing - allowed:
                self.fields.pop(field_name)  # удалили все поля кроме переданных
```
[к содержанию](#содержание)

Созданный сериализатор позволит выполнить следующее:

```python
 class UserSerializer(DynamicFieldsModelSerializer):
    # создадим новый класс, наследуясь от динамического
    # тк нету своего __init__, возьмет от динамического
     class Meta:
        model = User
        fields = ['id', 'username', 'email']  # existing

print(UserSerializer(user, fields=('id', 'email')))
# {'id': 2, 'email': 'jon@example.com'}
# тк поля переданы, то все остальные были удалены

print(UserSerializer(user))
# {'id': 2, 'username': 'jonwatts', 'email': 'jon@example.com'}
# тк поля не переданы, вернулись существующие
```

Такой способ позволяет динамически включать или исключать поля в сериализаторе, исходя из переданного списка fields. Если fields передан в конструктор, то только те поля, которые указаны в этом списке, будут сериализованы. Все остальные поля будут удалены из self.fields и не попадут в сериализованный результат.

### 1.7. Отношения сериализатора
Реляционные поля служат для представления отношений между моделями
(`ForeignKey`, `ManyToManyField` и `OneToOneField`, обратные и пользовательские отношения).

DRF не оптимизирует запросы по умолчанию, это делает разработчик сам.
Методы оптимизации запросов Django ORM, позволяют уменьшить количество запросов к бд и ускорить работу.
`select_related` — для связей ForeignKey и OneToOneField, когда нужно сделать JOIN и получить данные сразу.
`prefetch_related` — для связей ManyToManyField, reverse ForeignKey, или когда JOIN неудобен.

`reverse ForeignKey`: есть автор и книга. у книги внешний ключ на автора. тогда обратным доступом будет author.book_set.all(), related_name не указан

`select_related` получает за один запрос к бд все связанные данные, только для ForeignKey и OneToOneField:
```python
class Author(models.Model):
    name = models.CharField(max_length=100)

class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)

# Без select_related:
books = Book.objects.all()
for book in books:
    print(book.author.name)  # Каждый вызов book.author делает отдельный запрос

# С select_related:
books = Book.objects.select_related('author')
for book in books:
    print(book.author.name)  # Один запрос с JOIN, связанные данные получены сразу
```

`prefetch_related` - используется для выполнения двух отдельных запросов и связывания результатов в Python, для ManyToManyField и reverse ForeignKey связей:
```python
class Author(models.Model):
    name = models.CharField(max_length=100)

class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.ForeignKey(Author, on_delete=models.CASCADE, related_name='books')

# Без prefetch_related:
authors = Author.objects.all()
for author in authors:
    books = author.books.all()  # Отдельный запрос для каждого автора

# С prefetch_related:
authors = Author.objects.prefetch_related('books')
for author in authors:
    books = author.books.all()  # Один запрос для авторов и один для книг
```

пример с many=True:
```python
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.SlugRelatedField(
        many=True,
        read_only=True,
        slug_field='title'
    )

    class Meta:
        model = Album
        fields = ['album_name', 'artist', 'tracks']

# For each album object, tracks should be fetched from database
qs = Album.objects.all()
print(AlbumSerializer(qs, many=True).data)
```
Для каждого объекта `Album` будет выполняться отдельный запрос к базе данных для получения связанных объектов `tracks`. Если в базе данных, например, 100 альбомов, Django выполнит 101 запрос: 1 запрос для получения всех альбомов и еще 100 запросов для получения связанных треков каждого альбома.

```python
qs = Album.objects.prefetch_related('tracks')  # prefetch_related bcz it's reverse ForeignKey
# No additional database hits required
print(AlbumSerializer(qs, many=True).data)
```
Теперь Django выполнит 2 запроса вместо 101:
Один запрос для получения всех альбомов.
Один запрос для получения всех связанных треков сразу.
Django свяжет альбомы и их треки на уровне Python, избегая дополнительных обращений к базе данных при сериализации.

#### Проверка отношений
python manage.py shell
```python
>>> from myapp.serializers import AccountSerializer
>>> serializer = AccountSerializer()
>>> print(repr(serializer))
AccountSerializer():
    id = IntegerField(label='ID', read_only=True)
    name = CharField(allow_blank=True, max_length=100, required=False)
    owner = PrimaryKeyRelatedField(queryset=User.objects.all())
```

#### API Reference
```python
# модели для изучения реляционных полей
class Album(models.Model):
    album_name = models.CharField(max_length=100)
    artist = models.CharField(max_length=100)

class Track(models.Model):
    album = models.ForeignKey(Album, related_name='tracks', on_delete=models.CASCADE)
    order = models.IntegerField()
    title = models.CharField(max_length=100)
    duration = models.IntegerField()

    class Meta:
        unique_together = ['album', 'order']
        ordering = ['order']

    def __str__(self):
        return '%d: %s' % (self.order, self.title)
```

[к содержанию](#содержание)

##### StringRelatedField
`rest_framework.relations.StringRelatedField`

поле позволяет создавать строковое представление связанного объекта, используя метод `__str__()`

аргументы:
`many`.

```python
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.StringRelatedField(many=True)  # field is read only.

    class Meta:
        model = Album
        fields = ['album_name', 'artist', 'tracks']
```
```json
{
    "album_name": "Things We Lost In The Fire",
    "artist": "Low",
    "tracks": [
        "1: Sunflower",
        "2: Whitetail",
        "3: Dinosaur Act"
    ]
}
```

##### PrimaryKeyRelatedField
поле позволяет создавать и принимать pk связанного объекта.
для управления сериализацией/десериализацией значения первичного ключа. Например, pk_field=UUIDField(format='hex') сериализует pk в шестнадцатеричное представление.

аргументы:

`queryset`, `many`, `allow_null`,`pk_field`.



```python
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.PrimaryKeyRelatedField(many=True, read_only=True)

    class Meta:
        model = Album
        fields = ['album_name', 'artist', 'tracks']  # field is read-write
```
```json
{
    "album_name": "Undun",
    "artist": "The Roots",
    "tracks": [
        89,
        90,
        91
    ]
}

```

##### HyperlinkedRelatedField
поле позволяет создавать и принимать url связанного объекта.

аргументы:

`view_name`, `queryset`, `many`, `allow_null`, `lookup_field`, `lookup_url_kwarg`, `format`.

```python
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.HyperlinkedRelatedField(  # field is read-write
        many=True,
        read_only=True,
        view_name='track-detail'
    )

    class Meta:
        model = Album
        fields = ['album_name', 'artist', 'tracks']
```
```json
{
    "album_name": "Graceland",
    "artist": "Paul Simon",
    "tracks": [
        "http://www.example.com/api/tracks/45/",
        "http://www.example.com/api/tracks/46/",
        "http://www.example.com/api/tracks/47/"
    ]
}
```

##### SlugRelatedField
поле используется для представления связанного объекта с помощью определенного поля этого объекта.
отображает значение указанного поля.

аргументы:

`slug_field`, `queryset`, `many`, `allow_null`.


```python
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.SlugRelatedField(  # field is read-write
        many=True,
        read_only=True,
        slug_field='title'
     )

    class Meta:
        model = Album
        fields = ['album_name', 'artist', 'tracks']
```
```json
{
    "album_name": "Dear John",
    "artist": "Loney Dear",
    "tracks": [
        "Airport Surroundings",
        "Everything Turns to You",
        "I Was Only Going Out"
    ]
}
```
[к содержанию](#содержание)

##### HyperlinkedIdentityField
поле используется для создания ссылки, идентифицирующей объект. Оно автоматически генерирует URL для каждого экземпляра модели на основе указанного представления (view_name).

аргументы:

`view_name`, `lookup_field`, `lookup_url_kwarg`, `format`.

```python
class AlbumSerializer(serializers.HyperlinkedModelSerializer):
    track_listing = serializers.HyperlinkedIdentityField(view_name='track-list')  # field is read-only

    class Meta:
        model = Album
        fields = ['album_name', 'artist', 'track_listing']
```
```json
{
    "album_name": "The Eraser",
    "artist": "Thom Yorke",
    "track_listing": "http://www.example.com/api/track_list/12/"
}
```
опиши аргументы!!!

#### Вложенные отношения
Вложенные отношения реализуются с помощью вложенных сериализаторов, которые используются как поля в основном сериализаторе.
при этом данные связанного объекта оказываются вложенными в представление основного объекта.
Сериализатор вернет не ссылку на вложенный объект в виде его pk или url итд, а вставит всю информацию объекта (сериализует объект и вложит результат в соответствующее поле).

```python
class TrackSerializer(serializers.ModelSerializer):
    class Meta:
        model = Track
        fields = ['order', 'title', 'duration']

class AlbumSerializer(serializers.ModelSerializer):
    tracks = TrackSerializer(many=True, read_only=True)

    class Meta:
        model = Album
        fields = ['album_name', 'artist', 'tracks']
```
```python
>>> album = Album.objects.create(album_name="The Grey Album", artist='Danger Mouse')
>>> Track.objects.create(album=album, order=1, title='Public Service Announcement', duration=245)
<Track: Track object>
>>> Track.objects.create(album=album, order=2, title='What More Can I Say', duration=264)
<Track: Track object>
>>> Track.objects.create(album=album, order=3, title='Encore', duration=159)
<Track: Track object>
>>> serializer = AlbumSerializer(instance=album)
>>> serializer.data
```
```json
{
    "album_name": "The Grey Album",
    "artist": "Danger Mouse",
    "tracks": [
        {"order": 1, "title": "Public Service Announcement", "duration": 245},
        {"order": 2, "title": "What More Can I Say", "duration": 264},
        {"order": 3, "title": "Encore", "duration": 159}
    ]
}
```

По умолчанию вложенные сериализаторы read-only.
Чтобы вложенные сериализаторы могли изменять и создавать записи в бд, 
нужно создать им методы `create()` and/or `update()`, где указать, как нужно сохранить дочерние отношения.
```python
class TrackSerializer(serializers.ModelSerializer):
    class Meta:
        model = Track
        fields = ['order', 'title', 'duration']

class AlbumSerializer(serializers.ModelSerializer):
    tracks = TrackSerializer(many=True)

    class Meta:
        model = Album
        fields = ['album_name', 'artist', 'tracks']

    def create(self, validated_data):
        tracks_data = validated_data.pop('tracks')
        album = Album.objects.create(**validated_data)
        for track_data in tracks_data:
            Track.objects.create(album=album, **track_data)
        return album

>>> data = {
    'album_name': 'The Grey Album',
    'artist': 'Danger Mouse',
    'tracks': [
        {'order': 1, 'title': 'Public Service Announcement', 'duration': 245},
        {'order': 2, 'title': 'What More Can I Say', 'duration': 264},
        {'order': 3, 'title': 'Encore', 'duration': 159},
    ],
}
>>> serializer = AlbumSerializer(data=data)
>>> serializer.is_valid()  # True
>>> serializer.save()  # <Album: Album object>
```

#### Пользовательские реляционные поля

чтобы реализовать кастомное реляционное поле, нужно переопределить `RelatedField` и реализовать метод `.to_representation(self, value)`, где `value` экземпляр модели, который сериализуется.

Чтобы поле было read-write, нужно также реализовать метод ` .to_internal_value(self, data)`

Чтобы обеспечить динамический набор запросов, использующий `context`, нужно переопределить метод `.get_queryset(self)`, вместо выбора `.queryset` в классе или при инициализации поля.

```python
import time

class TrackListingField(serializers.RelatedField):
    def to_representation(self, value):
        duration = time.strftime('%M:%S', time.gmtime(value.duration))
        return 'Track %d: %s (%s)' % (value.order, value.name, duration)

class AlbumSerializer(serializers.ModelSerializer):
    tracks = TrackListingField(many=True)

    class Meta:
        model = Album
        fields = ['album_name', 'artist', 'tracks']
```
```json
{
    "album_name": "Sometimes I Wish We Were an Eagle",
    "artist": "Bill Callahan",
    "tracks": [
        "Track 1: Jim Cain (04:39)",
        "Track 2: Eid Ma Clack Shaw (04:19)",
        "Track 3: The Wind and the Dove (04:34)"
    ]
}
```
[к содержанию](#содержание)

#### Пользовательские поля с гиперссылками
если для URL-адресов требуется более одного поля поиска, нужно изменить поведение поля с гиперссылкой.

Для этого нужно переопределить `HyperlinkedRelatedField` и его методы:

`get_url(self, obj, view_name, request, format)`- используется для сопоставления объекта с его url.

`get_object(self, view_name, view_args, view_kwargs)` - чтобы поле с гиперссылкой было read-write, чтобы сопоставить полученный url с объектом, который url представляет.

```python
# /api/<organization_slug>/customers/<customer_pk>/


from rest_framework import serializers
from rest_framework.reverse import reverse

class CustomerHyperlink(serializers.HyperlinkedRelatedField):
    # We define these as class attributes, so we don't need to pass them as arguments.
    view_name = 'customer-detail'
    queryset = Customer.objects.all()

    def get_url(self, obj, view_name, request, format):
        url_kwargs = {
            'organization_slug': obj.organization.slug,
            'customer_pk': obj.pk
        }
        return reverse(view_name, kwargs=url_kwargs, request=request, format=format)

    def get_object(self, view_name, view_args, view_kwargs):
        lookup_kwargs = {
           'organization__slug': view_kwargs['organization_slug'],
           'pk': view_kwargs['customer_pk']
        }
        return self.get_queryset().get(**lookup_kwargs)
```
```text
Наследование от HyperlinkedRelatedField
Класс CustomerHyperlink наследуется от serializers.HyperlinkedRelatedField, 
чтобы создать поле, которое возвращает 
URL на объект Customer в сериализованном виде.

Определение view_name и queryset
view_name = 'customer-detail' — указывает на имя представления,
которое будет использоваться для генерации URL.
queryset = Customer.objects.all() — задает набор объектов, 
по которым будет выполняться поиск при валидации.

Метод get_url 
формирует URL для объекта Customer. 
Вместо использования только pk объекта, он добавляет два параметра:
organization_slug — slug организации, к которой принадлежит клиент.
customer_pk — первичный ключ клиента.
Метод вызывает функцию reverse, которая создает URL на основе 
имени представления (view_name) и переданных аргументов (url_kwargs).

Пример работы:
Если у объекта Customer:
organization.slug = "my-organization"
pk = 5
То метод сформирует URL примерно такого вида:
http://example.com/organizations/my-organization/customers/5/


Метод get_object
используется для получения объекта Customer при валидации 
входных данных (например, при обработке POST или PUT запросов).

view_kwargs содержит параметры, извлеченные из URL 
(например, organization_slug и customer_pk).
lookup_kwargs формируется для поиска объекта:
organization__slug — фильтрует по slug организации.
pk — фильтрует по первичному ключу клиента.
Метод вызывает get_queryset().get(**lookup_kwargs) 
для получения объекта Customer.
Пример работы:
Если клиент отправляет запрос с таким URL:
http://example.com/organizations/my-organization/customers/5/

То view_kwargs будет содержать:
{
    'organization_slug': 'my-organization',
    'customer_pk': 5
}
Метод выполнит запрос:
SELECT * FROM customer 
WHERE organization_slug = 'my-organization' AND id = 5;
```

#### Примечания

##### queryset
для полей отношений read-only queryset не нужен.
он нужен только для записываемых полей, тк определяет набор объектов для поиска при создании обновлении записей бд.
если поле записываемое - оно принимает входные данные, и queryset нужен для того чтобы в нем найти соответствующий входным данным объект.
сопоставить входные данные с объектом модели, а значит, корректно создать или обновить запись.

##### Настройка отображения HTML
Когда Django REST framework отображает HTML-форму в веб-интерфейсе API (например, для создания или обновления объекта), он автоматически создает выпадающий список (select input) для полей, которые являются отношениями (например, ForeignKey или ManyToMany).

Каждый элемент этого выпадающего списка (<option>) должен отображать строку, которую пользователь увидит и сможет выбрать. Эта строка — это название объекта, которое по умолчанию берется из метода __str__ модели, но его можно изменить, переопределив метод display_value().

в выпадающем списке(choices) используются строковые представления объектов (применяется метод __str__ модели).

Чтобы изменить это строковое представление, нужно переопределить `display_value()` подкласса `RelatedField`

`display_value()` получает на вход объект модели и возвращает соответствующее этому объекту строковое представление - строку, которая будет показана пользователю в выпадающем списке.
```python
class TrackPrimaryKeyRelatedField(serializers.PrimaryKeyRelatedField):
    def display_value(self, instance):
        return 'Track: %s' % (instance.title)
```
```html
<!--было, по умолчанию-->
<select name="tracks">
    <option value="1">Track 1</option>
    <option value="2">Track 2</option>
    <option value="3">Track 3</option>
</select>

<!--после переопределения display_value-->
<select name="tracks">
    <option value="1">Track: Track 1</option>
    <option value="2">Track: Track 2</option>
    <option value="3">Track: Track 3</option>
</select>


```

[к содержанию](#содержание)

##### Отсечение полей
Есть модели книга и автор. Книга связана с автором через ForeignKey.

При создании книги через Browsable API, увидим выпадающий список с авторами. Но если в базе данных более 1000 авторов, drf не будет пытаться отобразить всех, а покажет только сообщение: `"More than 1000 items…"`. 
Это нужно, чтобы уложиться в приемлемое время загрузки страницы.

`html_cutoff` - максимальное количество выборов, которое будет отображаться, по умолчанию 1000, None - нет ограничений.

`html_cutoff_text` - сообщение `"More than {count} items…"` отобразится если элементов больше, чем `html_cutoff`.

Изменить `html_cutoff` локально:
```python
class CustomPrimaryKeyRelatedField(serializers.PrimaryKeyRelatedField):
    def get_choices(self, cutoff=None):
        # Устанавливаем локальный лимит для этого поля
        cutoff = 500  # Или любое другое значение
        return super().get_choices(cutoff=cutoff)
```

Изменить `html_cutoff_text` локально:
```python
class CustomPrimaryKeyRelatedField(serializers.PrimaryKeyRelatedField):
    def get_choices(self, cutoff=None):
        if cutoff is None:
            cutoff = 1000  # Лимит элементов по умолчанию
        queryset = self.get_queryset()
        if queryset.count() > cutoff:
            return {'': 'Слишком много элементов... Введите ID вручную.'}
        return super().get_choices(cutoff=cutoff)
```

Применение в сериализаторе:
```python
class BookSerializer(serializers.ModelSerializer):
    author = CustomPrimaryKeyRelatedField(queryset=Author.objects.all())

    class Meta:
        model = Book
        fields = ['title', 'author']
```

Изменить глобально: `HTML_SELECT_CUTOFF` и `HTML_SELECT_CUTOFF_TEXT`.
```python
# settings.py
REST_FRAMEWORK = {
    'HTML_SELECT_CUTOFF': 500,  # Лимит отображаемых элементов
    'HTML_SELECT_CUTOFF_TEXT': 'Слишком много объектов... Введите ID вручную.'
}
```

Если ограничение есть, можно добавить в html форму поле ввода текста для поиска:
```python
assigned_to = serializers.SlugRelatedField(
   queryset=User.objects.all(),
   slug_field='username',
   style={'base_template': 'input.html'}
)
```

##### Обратные отношения
При создании сериализаторов ModelSerializer and HyperlinkedModelSerializer, Django автоматически добавляет только поля, описанные в модели - прямые отношения.

```python
# models.py
class Album(models.Model):
    name = models.CharField(max_length=100)  # прямое отношение

class Track(models.Model):
    title = models.CharField(max_length=100)
    album = models.ForeignKey(Album, on_delete=models.CASCADE)

# serializers.py
class AlbumSerializer(serializers.ModelSerializer):
    class Meta:
        model = Album
        fields = ['name']  # можно включить поле name для сериализации
```
Обратные отношения не включаются автоматически в ModelSerializer and HyperlinkedModelSerializer, их нужно явно добавить в список полей:
```python
class AlbumSerializer(serializers.ModelSerializer):
    class Meta:
        fields = ['tracks', ...]
```
поэтому для обратных отношений нужно установить такое связанное имя `related_name`, чтобы его можно было использовать как имя поля:
```python
class Track(models.Model):
    album = models.ForeignKey(Album, related_name='tracks', on_delete=models.CASCADE)
    ...
```
если `related_name` не установлено, то в сериализаторе нужно использовать автоматически сгенерированное имя:
```python
class AlbumSerializer(serializers.ModelSerializer):
    class Meta:
        fields = ['track_set', ...]
```
`related_name` — имя для обратного отношения в Django моделях.

##### Общие отношения
Generic Foreign Key (GFK) — это специальный тип связи, позволяющий модели ссылаться на разные типы объектов. 

В Django это достигается с помощью комбинации двух полей:
- `content_type` — указывает на тип связанного объекта.
- `object_id` — хранит идентификатор связанного объекта.

`GenericForeignKey` — объединяет эти два поля в одну связь.

Например, модель TaggedItem, связана с моделями Bookmark и Note,
Те содержит теги, которые можно привязывать как к закладкам (Bookmark), так и к заметкам (Note).
```python
class Bookmark(models.Model):
    """
    A bookmark consists of a URL, and 0 or more descriptive tags.
    """
    url = models.URLField()
    tags = GenericRelation(TaggedItem)


class Note(models.Model):
    """
    A note consists of some text, and 0 or more descriptive tags.
    """
    text = models.CharField(max_length=1000)
    tags = GenericRelation(TaggedItem)
    
    
class TaggedItem(models.Model):
    """
    Tags arbitrary model instances using a generic relation.

    See: https://docs.djangoproject.com/en/stable/ref/contrib/contenttypes/
    """
    tag_name = models.SlugField()
    content_type = models.ForeignKey(ContentType, on_delete=models.CASCADE)
    object_id = models.PositiveIntegerField()
    tagged_object = GenericForeignKey('content_type', 'object_id')

    def __str__(self):
        return self.tag_name
    
# content_type хранит тип связанного объекта (например, модель Article).
# object_id хранит ID конкретного объекта.
# content_object объединяет эти два поля, позволяя работать как с одним.
```

[к содержанию](#содержание)

По умолчанию DRF не знает, как сериализовать такие связи, потому что GenericForeignKey может ссылаться на объекты разных типов. Поэтому нужно определить кастомное поле, чтобы указать, как именно сериализовать связанные объекты.

Пример:
```python
bookmark = Bookmark(url='https://example.com')
tagged_item = TaggedItem(tag_name='Example Tag', tagged_object=bookmark)
```

```python
class TaggedObjectRelatedField(serializers.RelatedField):
    """
    A custom field to use for the `tagged_object` generic relationship.
    """

    def to_representation(self, value):
        """
        Serialize tagged objects to a simple textual representation.
        """
        if isinstance(value, Bookmark):
            return 'Bookmark: ' + value.url
        elif isinstance(value, Note):
            return 'Note: ' + value.text
        raise Exception('Unexpected type of tagged object')
```
```json
{
    "tag_name": "Example Tag",
    "tagged_object": "Bookmark: https://example.com"
}

```

То же, но для вложенного представления:
```python
def to_representation(self, value):
        """
        Serialize bookmark instances using a bookmark serializer,
        and note instances using a note serializer.
        """
        if isinstance(value, Bookmark):
            serializer = BookmarkSerializer(value)
        elif isinstance(value, Note):
            serializer = NoteSerializer(value)
        else:
            raise Exception('Unexpected type of tagged object')

        return serializer.data
```
```json
{
    "tag_name": "Example Tag",
    "tagged_object": {
        "url": "https://example.com"
    }
}
```

##### ManyToManyFields с промежуточной моделью
В Django связь `ManyToManyField` позволяет связывать две модели "многие ко многим". Обычно Django автоматически создаёт промежуточную (`through`) таблицу для такой связи. 

Если нужно добавить дополнительные поля в эту промежуточную таблицу, то нужно создать собственную `through` модель и ассоциировать ее с `ManyToManyField`, передав в аргумент `through`.


```python
class Membership(models.Model):
    # промежуточная модель, связывает Person и Group.
    # дополнительные полямя date_joined, role
    person = models.ForeignKey('Person', on_delete=models.CASCADE)
    group = models.ForeignKey('Group', on_delete=models.CASCADE)
    date_joined = models.DateField()
    role = models.CharField(max_length=50)

class Person(models.Model):
    name = models.CharField(max_length=100)
    groups = models.ManyToManyField('Group', through='Membership')

class Group(models.Model):
    name = models.CharField(max_length=100)
```

поля, которые ссылаются на ManyToManyField с through моделью, по умолчанию устанавливаются `read-only`. тк добавление и удаление объектов через такую связь требует явного управления данными промежуточной таблицы.

Как сериализовать ManyToManyField с through моделью?

Если нужно просто отобразить связанные объекты, без возможности их редактирования, можно явно указать поле в сериализаторе и установить read_only=True:
```python
class GroupSerializer(serializers.ModelSerializer):
    members = serializers.PrimaryKeyRelatedField(many=True, read_only=True, source='person_set')

    class Meta:
        model = Group
        fields = ['name', 'members']
```

Когда требуется представить дополнительные поля (date_joined, role) из промежуточной модели, можно создать отдельный сериализатор для этой модели и использовать её как вложенный объект:
```python
class MembershipSerializer(serializers.ModelSerializer):
    person_name = serializers.CharField(source='person.name')
    group_name = serializers.CharField(source='group.name')

    class Meta:
        model = Membership
        fields = ['person_name', 'group_name', 'date_joined', 'role']

# сериализатор для отображения данных Membership:
class GroupSerializer(serializers.ModelSerializer):
    memberships = MembershipSerializer(many=True, read_only=True, source='membership_set')

    class Meta:
        model = Group
        fields = ['name', 'memberships']
```

[к содержанию](#содержание)

### 1.8. Поля сериализатора

# 2. Routers
Роутеры автоматом связывают viewsets с url-адресами (создают маршруты для viewsets чтобы их не прописывать вручную в urlpatterns).
Могут использоваться и для APIView (через явную настройку маршрутов).
Основная задача роутеров - создавать маршруты для стандартных операций (создание, обновление, получение, удаление и т.д.)

```python
from rest_framework import routers

router = routers.SimpleRouter()  # создали экз маршрутизатора класса SimpleRouter
router.register(r'users', UserViewSet)  # зарегистрировали в нем view
router.register(r'accounts', AccountViewSet)
urlpatterns = router.urls  # созданный список маршрутов сделали основным, в который потом можно добавлять новые маршруты
```
аргументы для `register()`:
обязательные:
- `prefix` - префикс (например `r'users'`) урла для определенного набора маршрутов
- `viewset` - набор представлений для обработки сущности 
необязательные:
- `basename` - имя для урлов, автоматом генерируется из `queryset` (атрибут заданный во `viewset`), если нет `queryset`(переопределили `get_queryset`), то `basename` нужно явно задать.

`basename` это _начальная_ часть имени view, которая вставляется в урл:
- URL pattern: `^users/$` Name: `'user-list'` 
- URL pattern: `^users/{pk}/$` Name: `'user-detail'`

Атрибут .urls экземпляра маршрутизатора представляет собой просто стандартный список шаблонов URL-адресов. Существует несколько различных стилей включения этих URL-адресов.

## включение url адресов в список маршрутов
`router.urls` - это список маршрутов, которые были автоматически сгенерированы на основе зарегистрированных view.

Способы добавления новых маршрутов в основной список маршрутов `urlpatterns`:

```python
# добавление вручную: 
router = routers.SimpleRouter()
router.register(r'users', UserViewSet)
router.register(r'accounts', AccountViewSet)

urlpatterns = [
    path('forgot-password/', ForgotPasswordFormView.as_view()),
]

urlpatterns += router.urls  # сложение двух списков
```
```python
# джанговский include:
urlpatterns = [
    path('forgot-password', ForgotPasswordFormView.as_view()),
    path('', include(router.urls)),  
]
```
```python
# джанговский include с указанием namespace
urlpatterns = [
    path('forgot-password/', ForgotPasswordFormView.as_view()),
    path('api/', include((router.urls, 'app_name'))),
]
```
```python
# с указанием обоих namespace: приложения и экземпляра
urlpatterns = [
    path('forgot-password/', ForgotPasswordFormView.as_view()),
    path('api/', include((router.urls, 'app_name'), namespace='instance_name')),
]
```
Для Hyperlinked serializers:
Hyperlinked serializers в DRF автоматически генерируют ссылки на представления, используя значение view_name.
```python
class UserSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = User
        fields = ['url', 'username', 'email']
# По умолчанию, DRF автоматически определяет view_name, 
# используя шаблон %(model_name)s-detail. 
# В этом примере DRF ожидает маршрут с именем 'user-detail'. 
# Но если маршруты были объявлены с namespace, 
# тк имя маршрута изменяется с user-detail на 'app_name:user-detail'. 
# нужно вручную указать полное имя маршрута, например:
url = serializers.HyperlinkedIdentityField(view_name='app_name:user-detail')
```

namespace что это и зачем:
нужен, чтобы различать маршруты с одинаковыми именами, если они находятся в разных приложениях. 
```python
# urls.py в приложении 'app_name'
app_name = 'app_name'  # задали namespace в приложении

urlpatterns = [
    path('users/<int:pk>/', UserDetailView.as_view(), name='user-detail'),
]
# Когда в основном urls.py этот набор маршрутов подключается 
# с пространством имен, он будет доступен как 
# app_name:user-detail, а не просто user-detail
```
```python
url = serializers.HyperlinkedIdentityField(view_name='app_name:user-detail')
```

## Маршрутизация дополнительных действий добавленных во viewset
```python
from myapp.permissions import IsAdminOrIsSelf
from rest_framework.decorators import action

class UserViewSet(ModelViewSet):
    ...

    @action(methods=['post'], detail=True, permission_classes=[IsAdminOrIsSelf])
    def set_password(self, request, pk=None):
        ...
# через @action во viewset добавляются дополнительные действия, 
# для которых автоматом генерируются маршруты.
```
Шаблон URL: `^users/{pk}/set_password/$`
Имя URL: `'user-set-password'`

По умолчанию шаблон URL генерируется на основе имени метода, а имя маршрута создается как комбинация basename ViewSet'а и имени метода, записанного через дефис.
Если вы хотите использовать собственные значения для имени маршрута или URL, можно передать аргументы url_path и url_name в декоратор @action.
```python
from myapp.permissions import IsAdminOrIsSelf
from rest_framework.decorators import action

class UserViewSet(ModelViewSet):
    ...

    @action(methods=['post'], detail=True, permission_classes=[IsAdminOrIsSelf],
            url_path='change-password', url_name='change_password')
    def set_password(self, request, pk=None):
        ...
```
URL path: `^users/{pk}/change-password/$`
URL name: `'user-change_password'`

## SimpleRouter
автоматически создает маршруты для стандартного набора действий: list, create, retrieve, update, partial_update и destroy, но не создает корневой путь api:
```python
# по умолчанию URLs, создаваемые SimpleRouter, заканчиваются на /,
# чтобы изменить это:
router = SimpleRouter(trailing_slash=False)
```

```python
# По умолчанию SimpleRouter использует регулярные выражения для маршрутов,
# чтобы изменить это:
router = SimpleRouter(use_regex_path=False)
# но только в джанге 2.x и выше
```
в урле могут быть использованы все значения кроме слэша и точки.
чтобы больше ограничить возможные значения можно использовать:
```python
class MyModelViewSet(mixins.RetrieveModelMixin, viewsets.GenericViewSet):
    lookup_field = 'my_model_id'  # поле для поиска значения по бд( вместо дефолтного pk)
    lookup_value_regex = '[0-9a-f]{32}'  # regex определяет, какие значения могут использоваться в URL

class MyPathModelViewSet(mixins.RetrieveModelMixin, viewsets.GenericViewSet):
    lookup_field = 'my_model_uuid'
    lookup_value_converter = 'uuid'  # преобразователь пути uuid проверяет, что значение в URL является корректным UUID 
                                     # формата 8-4-4-4-12 (например, 123e4567-e89b-12d3-a456-426614174000)
```

## DefaultRouter
Генерирует стандартные пути + включает корневой путь api:
```python
from rest_framework.routers import DefaultRouter
from myapp.views import UserViewSet

router = DefaultRouter()
router.register(r'users', UserViewSet)
```

## Custom Routers

# Authentication

- _Аутентификация_ — (кто?) это процесс идентификации пользователя системой в которую он входит.
- _Авторизация_ — (что?) это процесс, проверяющий, имеет ли пользователь права на выполнения действия.

Аутентификация - это механизм связывания входящего запроса с набором идентификационных данных.
Выполняется в самом начале обработки представления, до того как происходят проверки разрешений и ограничений
Схема аутентификации - это список классов. С каждым из них drf будет пытаться идентифицировать запрос. Значения request.user и request.auth установятся классом, который выполнит аутентификацию успешно.

## Установка схемы аутентификации
глобально:
```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.BasicAuthentication',
        'rest_framework.authentication.SessionAuthentication',
    ]
}
```
локально, для каждого представления:
```python
class ExampleView(APIView):
    authentication_classes = [SessionAuthentication, BasicAuthentication]
    permission_classes = [IsAuthenticated]

    def get(self, request, format=None):
        content = {
            'user': str(request.user),  # `django.contrib.auth.User` instance.
            'auth': str(request.auth),  # None
        }
        return Response(content)

@api_view(['GET'])
@authentication_classes([SessionAuthentication, BasicAuthentication])
@permission_classes([IsAuthenticated])
def example_view(request, format=None):
    content = {
        'user': str(request.user),  # Экземпляр `django.contrib.auth.User`.
        'auth': str(request.auth),  # None
    }
    return Response(content)
```
ошибки:

- HTTP 401 Unauthorized
- HTTP 403 Permission Denied

Ответы HTTP 401 всегда должны включать заголовок `WWW-Authenticate`, который подсказывает клиенту, как аутентифицироваться. Ответы HTTP 403 не включают заголовок `WWW-Authenticate`.

## Виды аутентификации в DRF

## BasicAuthentication
подходит только для тестирования.
Basic Authentication передает имя пользователя и пароль в открытом виде (хотя и закодированные в base64), что делает этот метод небезопасным при передаче по незащищенному каналу (например, HTTP). Поэтому его следует использовать только с HTTPS, чтобы избежать перехвата данных.
Также необходимо убедиться, что клиенты API всегда будут запрашивать имя пользователя и пароль при входе в систему и никогда не будут хранить эти данные в постоянном хранилище.
```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.BasicAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',  # Требуется авторизация для доступа
    ],
}
```
можно комбинировать:
```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.BasicAuthentication',    # Для тестирования и отладки
        'rest_framework.authentication.TokenAuthentication',    # Основной метод для продакшена
        'rest_framework.authentication.SessionAuthentication',  # Для веб-интерфейса с авторизацией через сессии
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',  # Требуется аутентификация для всех запросов
    ],
}
```

## Token Authentication

Аутентификация через токены

Как работает:

Token Authentication — это механизм, при котором каждый запрос от клиента включает токен в заголовке авторизации (обычно в виде Authorization: Token <token>). Токен обычно выдается пользователю при его успешной аутентификации (например, при регистрации или логине).

Настройка:

configure the authentication classes to include TokenAuthentication
```python
INSTALLED_APPS = [
    'rest_framework.authtoken'
]
```
make manage.py migrate

create token for users:
```python
from rest_framework.authtoken.models import Token

token = Token.objects.create(user=...)
print(token.key)
```
now token key should be included in the Authorization HTTP header as `Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b`

### генерация токенов
с использованием сигналов:
```python
# models.py:
from django.conf import settings
from django.db.models.signals import post_save
from django.dispatch import receiver
from rest_framework.authtoken.models import Token

@receiver(post_save, sender=settings.AUTH_USER_MODEL)
def create_auth_token(sender, instance=None, created=False, **kwargs):
    if created:
        Token.objects.create(user=instance)
```
```python
# генерировать токен для уже существующего юзера:
from django.contrib.auth.models import User
from rest_framework.authtoken.models import Token

for user in User.objects.all():
    Token.objects.get_or_create(user=user)
```

предоставляя API endpoint:
```python
# urls.py:
from rest_framework.authtoken import views
urlpatterns += [
    path('api-token-auth/', views.obtain_auth_token)
]
```

используя джанго админку:
```python
# admin.py:
from rest_framework.authtoken.admin import TokenAdmin

TokenAdmin.raw_id_fields = ['user']
```

испльзуя команду manage.py:

`/manage.py drf_create_token <username>`

return `Generated token 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b for user user1`

чтобы восстановить токен:

`./manage.py drf_create_token -r <username>`

Отличия аутентификации по дрф токенам от JWT:

по токенам дрф: 
клиент регистрируется, если пароль, логин, емайл валидны - сервер сохраняет юзера в бд, 
юзер логинится, сервер проверяет, если все корректно, генерирует токен, сохраняет его в бд и отдает клиенту, в следующем запросе клиент передает токен в своем заголовке, сервер берет его и ищет в бд связанного с этим токеном клиента и если все хорошо, обрабатывает запрос или возвращает `401 Unauthorized`

по JWT:
клиент регистрируется, если пароль, логин, емайл валидны - сервер сохраняет юзера в бд, юзер логинится, сервер проверяет, если все корректно, генерирует два токена: access and refresh и возвращает их клиенту, у себя их не хранит. в следующем запросе клиент передает access-токен в заголовке своего запроса, сервер проверяет подпись токена и срок его действия, если все хорошо - сервер обрабатывает запрос. сервер проверяет токен с помощью криптографической подписи, не обращаясь к базе. 

Обновление токенов:
токен дрф по умолчанию не имеет срока действия, если сервер его удалил, нужен повторный логин.
у JWT access-токен имеет короткий срок действия, при его истечении клиент использует refresh-токен для получения нового access-токена

Итого:

DRF Token — сравнивает токен с записями в базе данных.
JWT — проверяет подпись и срок действия токена без обращения к базе данных.

## Session Authentication   
используется по умолчанию

настройка:
```python
# settings.py:

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
    ],
}
```
для сессионной аутентификации в drf создана форма ввода login/logout.

подключить форму:
```python
# urls.py:
urlpatterns = [
    path('api-auth/', include('rest_framework.urls', namespace='rest_framework')),
]
```
Как работает:

Когда пользователь входит в систему (через форму или API), Django создает сессию и сохраняет её идентификатор в cookie. В каждом последующем запросе браузер отправляет cookie с сессионным ID, который сервер использует для идентификации пользователя. Если сессионный ID в cookie совпадает с сессионным ID, хранящимся на сервере, и сессия действительна, пользователь считается аутентифицированным.

Где и зачем использовать:
- Подходит для веб-приложений, с формами для входа, т к сессия позволяет серверу "запомнить" пользователя, чтобы ему не приходилось повторно вводить логин и пароль на каждой странице. 

- Django автоматически использует сессию для проверки CSRF (защиты от межсайтовой подделки запросов).

- В традиционных веб-приложениях, где страницы обновляются полностью, а не динамически через AJAX, сессии позволяют "запомнить" пользователя между загрузками страниц.

По умолчанию для совместимости с веб-приложениями, созданными на базе Django, где сессии являются стандартом. Однако для RESTful API, работающих с внешними клиентами, лучше использовать токен- или JWT-аутентификацию, чтобы соблюдать принцип «stateless».

# Permissions
виды прав: 
  -  `AllowAny`
  -  `IsAuthenticated`
  -  `IsAdminUser`
  -  `IsAuthenticatedOrReadOnly`
  -  `DjangoModelPermissions`
  -  `DjangoModelPermissionsOrAnonReadOnly`
  -  `DjangoObjectPermissions`

настройка глобально:
```python
# settings.py:
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
        # 'rest_framework.permissions.AllowAny' - по умолчанию разрешено всем
    ]
}
```
локально, переопределит глобальную настройку для этого view:
```python
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework.views import APIView
from rest_framework.decorators import api_view, permission_classes


class ExampleView(APIView):
    permission_classes = [IsAuthenticated]

    def get(self, request, format=None):
        content = {
            'status': 'request was permitted'
        }
        return Response(content)

    
@api_view(['GET'])
@permission_classes([IsAuthenticated])
def example_view(request, format=None):
    content = {
        'status': 'request was permitted'
    }
    return Response(content)
```

кастомные разрешения

нужно унаследоваться от `BasePermission` и переопределить методы `.has_permission` и/или `.has_object_permission`, где:

- `has_object_permission(self, request, view, obj)` - вызывается для каждого объекта и проверяет доступ к нему.
- `has_permission(self, request, view)` - проверяет есть ли права доступа ко всему представлению, может ли пользователь вообще выполнять запрос к данному API-endpoint(url).
```python
from rest_framework.permissions import BasePermission

class IsAdminUser(BasePermission):
    def has_permission(self, request, view):
        # проверяет, что пользователь аутентифицирован (request.user существует) 
        # и имеет статус администратора (is_staff)
        return request.user and request.user.is_staff
```
```python
from rest_framework.permissions import BasePermission

class IsOwner(BasePermission):
    def has_object_permission(self, request, view, obj):
        # Доступ разрешён только если пользователь является владельцем объекта
        # чтобы изменяли только свое 
        return obj.owner == request.user
```
```python
from rest_framework.permissions import BasePermission, SAFE_METHODS

class ReadOnlyOrAuthenticated(BasePermission):
    def has_permission(self, request, view):
        # Доступ к чтению разрешён всем
        if request.method in SAFE_METHODS:  
        # если в списке безопасных методов HTTP: GET, HEAD, OPTIONS
            return True
        # если в списке небезопасных: POST, PUT, DELETE
        # Доступ к записи разрешён только аутентифицированным пользователям
        return request.user and request.user.is_authenticated
```
```python
from rest_framework.permissions import BasePermission

class IsManagerOrOwner(BasePermission):
    def has_object_permission(self, request, view, obj):
        # Менеджеры имеют доступ ко всем объектам
        if request.user.role == 'manager':
            return True
        # Владелец имеет доступ только к своим объектам
        return obj.owner == request.user
```

объединение разрешений
```python
class ReadOnly(BasePermission):
    def has_permission(self, request, view):
        return request.method in SAFE_METHODS

class ExampleView(APIView):
    permission_classes = [IsAuthenticated|ReadOnly]  # можно если ReadOnly наследовался от BasePermission

    def get(self, request, format=None):
        content = {
            'status': 'request was permitted'
        }
        return Response(content)

# it supports & (and), | (or) and ~ (not).
# здесь | используется для создания объединённого разрешения, 
# которое пропускает запрос, если хотя бы одно из разрешений возвращает True
```
DRF поддерживает _объединение разрешений_ с помощью побитовых операций (&, |, ~).
(классы разрешений реализуют методы `__or__`, `__and__`, `__invert__`).
Использование | упрощает код, делая его более читаемым и декларативным.

Чтобы написать то же без использования |, придется создать кастомное разрешение вручную:

```python
class CustomPermission(BasePermission):
    def has_permission(self, request, view):
        return request.user.is_authenticated or request.method in SAFE_METHODS
```

```python
# & - если нужно разрешить доступ, когда оба разрешения возвращают True:
permission_classes = [IsAuthenticated & SomeOtherPermission]
```
```python
# ~ (НЕ) - Инвертирует разрешение:
permission_classes = [~IsAuthenticated]  # доступ только для неаутентифицированных пользователе
```

## Validators

В drf валидация выполняется полностью в сериализаторах.

Если используется класс `Serializer`, можно повторить поведение тех же валидаторов, что используются в `ModelSerializer`, отличие в том, что в `ModelSerializer` валидаторы срабатывают автоматически, а в `Serializer` их нужно прописать явно.

Все валидаторы которые используются для поля можно посмотреть через `repr(instance_serializer)`
```python
# models.py:
class CustomerReportRecord(models.Model):
    time_raised = models.DateTimeField(default=timezone.now, editable=False)
    reference = models.CharField(unique=True, max_length=20)
    description = models.TextField()

# serializers.py:
class CustomerReportSerializer(serializers.ModelSerializer):
    class Meta:
        model = CustomerReportRecord

# python console:
>>> from project.example.serializers import CustomerReportSerializer
>>> serializer = CustomerReportSerializer()
>>> print(repr(serializer))

# output:
CustomerReportSerializer():
    id = IntegerField(label='ID', read_only=True)
    time_raised = DateTimeField(read_only=True)
    reference = CharField(max_length=20, validators=[<UniqueValidator(queryset=CustomerReportRecord.objects.all())>])
    description = CharField(style={'type': 'textarea'})
```
смотреть как на поле reference сработал валидатор уникальности. Но это валидатор drf, а джанговский валидатор сработавший на том же поле на max_length не показался...

### Виды валидаторов DRF

`UniqueValidator`

проверяет что поле уникально, принимает:

`queryset` - required, объекты поля с таким ограничением.

`message`  - сообщение об ошибке.

`lookup ` - тип поиска, by default 'exact' - точное совпадение

```python
from rest_framework.validators import UniqueValidator

slug = SlugField(
    max_length=100,
    validators=[UniqueValidator(queryset=BlogPost.objects.all())]
  
# validators=[UniqueValidator(queryset=User.objects.all(), lookup='iexact')]
  
# 'iexact' - ищет без учета регистра.
# 'contains' — содержится ли значение в поле.
# 'icontains' — содержится ли значение в поле, без учета регистра.
# 'gt', 'lt', 'gte', 'lte' — для числовых значений, сравнивая их с полем.
)
```

`UniqueTogetherValidator`

проверяет, что поля уникальны в своей комбинации, принимает:

`queryset` - required, объекты поля с таким ограничением.

`fields` - required, поля для сериализации.

`message` - необязательный - сообщение об ошибке когда валидация провалилась

```python
from rest_framework.validators import UniqueTogetherValidator

class ExampleSerializer(serializers.Serializer):
    # ...
    class Meta:
        # ToDo items belong to a parent list, and have an ordering defined
        # by the 'position' field. No two items in a given list may share
        # the same position.
        validators = [
            UniqueTogetherValidator(
                queryset=ToDoItem.objects.all(),
                fields=['list', 'position']
            )
        ]
```
если поле с ограничением уникальности может принимать значение по умолчанию, то встретив такое значение `UniqueValidator` не будет его райзить.

`UniqueForDateValidator`
`UniqueForMonthValidator`
`UniqueForYearValidator`
проверяет, что поля уникальны за период (день, месяц, год), принимает:

`queryset` - required, объекты поля с таким ограничением.

`field` - required, имя поля которое должно быть уникальным.

`date_field` - required, поле дат, по которому будет выбираться заданный диапазон.

`message` - сообщение об ошибке когда валидация провалилась

```python
from rest_framework.validators import UniqueForYearValidator

class ExampleSerializer(serializers.Serializer):
    # ...
    class Meta:
        # все посты где slug уникален в течение этого года.
        validators = [
            UniqueForYearValidator(
                queryset=BlogPostItem.objects.all(),
                field='slug',
                date_field='published'
            )
        ]
```
```python
published = serializers.DateTimeField(required=True)  # for writable date_field
published = serializers.DateTimeField(read_only=True, default=timezone.now)  # for read-only date_field
published = serializers.HiddenField(default=timezone.now)  # hidden date_field
# HiddenField не принимает пользовательский ввод, 
# но всегда возвращает значение по умолчанию в поле validated_data в сериализаторе.
# HiddenField() does not appear in partial=True serializer(PATCH request)
```

- Поле `date_field` обязательно для сериализации.

- Если в модели для этого поля задано значение по умолчанию, его нужно явно указать в сериализаторе, на случай если пользователь не введет дату. 

- Так как значение по умолчанию подставляется после валидации, сериализатор должен получить поле до валидации, чтобы корректно выполнить валидацию.

- Поэтому нужно либо явно передать сериализатору значение по умолчанию из модели, либо заставить пользователя всегда вводить дату:
```python
class Event(models.Model):
    name = models.CharField(max_length=100)
    date = models.DateField(default=now)

class EventSerializer(serializers.ModelSerializer):
    date = serializers.DateField(required=True)  # заставит юзера всегда вводить
    # date = serializers.DateField()  # подставит значение по умолчанию из модели

    class Meta:
        model = Event
        fields = ['name', 'date']
```

HiddenField - поля для скрытых данных.

для выполнения валидации могут понадобиться скрытые поля. `HiddenField` - это тип полей, доступ к которым имеет только внутренняя логика, но не пользователь API.
Значение полей HiddenField будет в `validated_data`, но не среди сериализованных данных.
Используется для передачи данных внутри сериализатора.

`read_only=True` не будет использовать аргумент `default=`..., потому что оно предназначено только для чтения.

С полями `HiddenField` часто используются классы `CurrentUserDefault` и `CreateOnlyDefault`.

- `CurrentUserDefault`
устанавливает текущее значение пользователя (`request.user`) в поле. Для этого нужно передать объект `request` в контексте сериализатора.
```python
from rest_framework import serializers

class PostSerializer(serializers.ModelSerializer):
    owner = serializers.HiddenField(
        default=serializers.CurrentUserDefault()  
        # Устанавливает текущего пользователя как владельца
    )
    
    class Meta:
        model = Post
        fields = ['title', 'content', 'owner']

# при создании экз сериализатора, передать контекстный словарь:
serializer = PostSerializer(data=request_data, context={'request': request})
```

- `CreateOnlyDefault`
устанавливает значение по умолчанию только при создании объекта. При обновлениях это поле игнорируется.

```python
from rest_framework import serializers
from django.utils import timezone

class PostSerializer(serializers.ModelSerializer):
    created_at = serializers.DateTimeField(
        default=serializers.CreateOnlyDefault(timezone.now)  
        # Устанавливает текущую дату и время только при создании, но не при обновлении
    )
    
    class Meta:
        model = Post
        fields = ['title', 'content', 'created_at']

```

### Ограничения валидаторов в Django REST Framework

- Неопределённые случаи и явная валидация: 

Когда стандартные валидаторы, генерируемые `ModelSerializer`, не подходят, требуется ручная валидация.

Для этого нужно отключить их, указав пустой список для `validators` в классе `Meta`.

```python
class BillingRecordSerializer(serializers.ModelSerializer):
    def validate(self, attrs):
        # Ваша кастомная валидация
        return attrs

    class Meta:
        fields = ['client', 'date', 'amount']
        extra_kwargs = {'client': {'required': False}}
        validators = []  # Убираем стандартные валидаторы
```

- Необязательные поля и уникальность: 

При использовании `unique_together` все поля, входящие в это ограничение, по умолчанию считаются обязательными. Если одно из полей должно быть необязательным, валидация становится неопределенной.

Чтобы это исправить нужно убрать валидатор из сериализатора и написать кастомную логику в `.validate()` или в представлении.
```python
Копировать код
class BillingRecordSerializer(serializers.ModelSerializer):
    class Meta:
        fields = ['client', 'date', 'amount']
        extra_kwargs = {'client': {'required': False}}
        validators = []  # Отключаем стандартную валидацию уникальности
```

- Обновление вложенных сериализаторов: 

При обновлении экземпляра объекта стандартные валидаторы уникальности исключают текущий экземпляр из проверки, чтобы с ним не было конфликта. Для вложенных сериализаторов это не применимо, тк текущий экземпляр не доступен для проверки, тк экземпляр родительского объекта не передаётся в сериализатор вложенного объекта.

Чтобы это исправить нужно написать кастомную валидацию, где получить доступ к текущему экземпляру и исключить его из проверки уникальности.
```python
# models:
class Author(models.Model):
    name = models.CharField(max_length=100)

class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.ForeignKey(Author, related_name='books', on_delete=models.CASCADE)


class AuthorSerializer(serializers.ModelSerializer):
    class Meta:
        model = Author
        fields = ['name']

# serializers:
class BookSerializer(serializers.ModelSerializer):
    author = AuthorSerializer()

    class Meta:
        model = Book
        fields = ['title', 'author']

    def validate_title(self, value):
        # Если мы обновляем объект, исключаем текущий экземпляр из проверки уникальности
        book_instance = self.instance  # Текущий экземпляр (который обновляется)
        if book_instance:
            # Исключаем текущий экземпляр из проверки уникальности
            if Book.objects.filter(title=value).exclude(id=book_instance.id).exists():
                raise serializers.ValidationError("This title is already taken.")
        else:
            # Если объект новый (создаётся), просто проверяем уникальность
            if Book.objects.filter(title=value).exists():
                raise serializers.ValidationError("This title is already taken.")
        
        return value
```

- Отладка сложных случаев: 

Чтобы понять, как `ModelSerializer` генерирует валидаторы и поля, нужно использовать `manage.py shell`, чтобы вывести сериализатор и изучить его.
```python
serializer = MyComplexModelSerializer()
print(serializer)
```
Для сложных случаев лучше явно определить сериализатор вместо того, чтобы полагаться на стандартное поведение `ModelSerializer`, чтобы поведение было более понятным.



### Кастомные валидаторы

- функции-валидаторы:

```python
# создание:
def even_number(value):
    if value % 2 != 0:
        raise serializers.ValidationError('This field must be an even number.')
    
# применение:
class MySerializer(serializers.Serializer):
    number = serializers.IntegerField(validators=[even_number])
```

- валидаторы уровня поля:

```python
class MySerializer(serializers.Serializer):
    number = serializers.IntegerField()

    def validate_number(self, value):
        # DRF будет вызывать метод автоматически только для поля number на основе его имени.
        if value % 2 != 0:
            raise serializers.ValidationError('This field must be an even number.')
        return value
```

- классы-валидаторы:

```python
# создание:
class MultipleOf:
    def __init__(self, base):
        self.base = base

    def __call__(self, value):
        if value % self.base != 0:
            message = 'This field must be a multiple of %d.' % self.base
            raise serializers.ValidationError(message)

# применение:
class MySerializer(serializers.Serializer):
    number = serializers.IntegerField(validators=[MultipleOf(3)])
```

Доступ к контексту

Если указать `requires_context = True`, метод `__call__` валидатора будет вызываться с дополнительным аргументом `serializer_field`, который даёт доступ к полю сериализатора.
```python
class MultipleOf:
    requires_context = True

    def __call__(self, value, serializer_field):
        # использование serializer_field для проверки,
        # является ли поле обязательным, например.
        if value % 3 != 0:
            raise serializers.ValidationError('This field must be a multiple of 3.')
# где: 
# value — значение поля, которое нужно проверить
# serializer_field — объект, представляющий поле, для которого выполняется валидация.
```















### 2. **Как работают ссылки**
- Заголовки в Markdown автоматически превращаются в якоря, к которым можно переходить с помощью ссылок.
- Ссылка вида `[Сериализация (Serialization)](#сериализация-serialization)` перемещает пользователя к разделу `## Сериализация (Serialization)`.

---

### 3. **Добавление ссылки "Назад к содержанию"**
Каждый раздел заканчивается ссылкой `[К содержанию](#содержание)`, чтобы пользователь мог быстро вернуться к началу.

---

### 4. **Предпросмотр файла**
- Откройте файл `README.md` в PyCharm.
- Нажмите на значок глазика в правом верхнем углу, чтобы включить предпросмотр Markdown.

---




