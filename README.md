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
     - [API field_class and field_kwargs](#api-field-class-and-field-kwargs)






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
Если поле, которое должно быть только для чтения, является частью ограничение `unique_together`

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

#### API field_class и field_kwargs
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

### HyperlinkedModelSerializer

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




