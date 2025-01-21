# Основные команды Django ORM

Django ORM (Object-Relational Mapping) предоставляет удобный способ работы с базами данных, используя Python-код вместо SQL-запросов. Вот основные команды и методы, которые используются при работе с моделями Django.

---

## Содержание

- [1. Создание объектов](#1-создание-объектов)
- [2. Чтение объектов (Запросы)](#2-чтение-объектов-запросы)
  - [Получение всех объектов](#получение-всех-объектов)
  - [Фильтрация](#фильтрация)
  - [Получение одного объекта](#получение-одного-объекта)
  - [Ленивая инициализация](#ленивая-инициализация)
- [3. Обновление объектов](#3-обновление-объектов)
  - [Изменение полей существующего объекта](#изменение-полей-существующего-объекта)
  - [Массовое обновление](#массовое-обновление)
- [4. Удаление объектов](#4-удаление-объектов)
  - [Удаление одного объекта](#удаление-одного-объекта)
  - [Массовое удаление](#массовое-удаление)
- [5. Сложные запросы](#5-сложные-запросы)
  - [Сортировка](#сортировка)
  - [Ограничение количества результатов](#ограничение-количества-результатов)
  - [Агрегации](#агрегации)
- [6. Связи между моделями](#6-связи-между-моделями)
  - [Получение связанных объектов](#получение-связанных-объектов)
- [7. Raw SQL (При необходимости)](#7-raw-sql-при-необходимости)
- [8. Создание сложных запросов](#8-создание-сложных-запросов)
  - [Использование Q-объектов для сложных условий](#использование-q-объектов-для-сложных-условий)
- [9. Менеджеры моделей](#9-менеджеры-моделей)
  - [Создание собственного менеджера](#создание-собственного-менеджера)
- [10. Транзакции](#10-транзакции)

---

## 1. Создание объектов

Создание и сохранение нового объекта в базе данных:

```python
from myapp.models import MyModel

# Создание объекта
obj = MyModel(field1='value1', field2='value2')
obj.save()  # Сохранение объекта в базе данных

# Или создание и сохранение в одну строку
obj = MyModel.objects.create(field1='value1', field2='value2')
```

---

## 2. Чтение объектов (Запросы)

### Получение всех объектов:
```python
all_objects = MyModel.objects.all()
```

### Фильтрация:
```python
# Получение объектов, соответствующих условиям
filtered_objects = MyModel.objects.filter(field1='value1')

# Исключение объектов, соответствующих условиям
excluded_objects = MyModel.objects.exclude(field1='value1')
```

### Получение одного объекта:
```python
# Если объект существует
single_object = MyModel.objects.get(id=1)

# Если объекта может не быть
from django.core.exceptions import ObjectDoesNotExist
try:
    single_object = MyModel.objects.get(id=1)
except ObjectDoesNotExist:
    single_object = None
```

### Ленивая инициализация:
Запросы в Django ORM выполняются лениво. Например, `MyModel.objects.all()` не выполняет запрос до момента его использования (например, в `for`-цикле).

---

## 3. Обновление объектов

### Изменение полей существующего объекта:
```python
obj = MyModel.objects.get(id=1)
obj.field1 = 'new_value'
obj.save()
```

### Массовое обновление:
```python
MyModel.objects.filter(field1='value1').update(field2='new_value')
```

---

## 4. Удаление объектов

### Удаление одного объекта:
```python
obj = MyModel.objects.get(id=1)
obj.delete()
```

### Массовое удаление:
```python
MyModel.objects.filter(field1='value1').delete()
```

---

## 5. Сложные запросы

### Сортировка:
```python
sorted_objects = MyModel.objects.order_by('field1')  # По возрастанию
sorted_objects_desc = MyModel.objects.order_by('-field1')  # По убыванию
```

### Ограничение количества результатов:
```python
limited_objects = MyModel.objects.all()[:10]  # Первые 10 объектов
```
[к содержанию](#содержание)

### Агрегации:
```python
from django.db.models import Count, Sum, Avg, Max, Min

# Пример: Подсчитать количество записей
count = MyModel.objects.count()

# Пример: Подсчитать суммы или средние значения
aggregates = MyModel.objects.aggregate(total=Sum('field1'), average=Avg('field2'))
```

---

## 6. Связи между моделями

### Получение связанных объектов:

- Для полей `ForeignKey`:
```python
parent = ParentModel.objects.get(id=1)
children = parent.childmodel_set.all()  # Все связанные объекты
```

- Для полей `ManyToManyField`:
```python
obj = MyModel.objects.get(id=1)
related_objects = obj.related_field.all()
```

---

## 7. Raw SQL (При необходимости)
Иногда нужно выполнить "сырые" SQL-запросы:
```python
from django.db import connection

with connection.cursor() as cursor:
    cursor.execute("SELECT * FROM myapp_mymodel WHERE field1 = %s", ['value1'])
    rows = cursor.fetchall()
```

---

## 8. Создание сложных запросов

### Использование Q-объектов для сложных условий:
```python
from django.db.models import Q

# Пример: OR-условия
objects = MyModel.objects.filter(Q(field1='value1') | Q(field2='value2'))

# Пример: NOT-условия
objects = MyModel.objects.filter(~Q(field1='value1'))
```

---

## 9. Менеджеры моделей

### Создание собственного менеджера:
Менеджеры позволяют добавлять свои методы запросов.

```python
from django.db import models

class MyModelManager(models.Manager):
    def active(self):
        return self.filter(is_active=True)

class MyModel(models.Model):
    field1 = models.CharField(max_length=100)
    is_active = models.BooleanField(default=True)

    objects = MyModelManager()  # Назначение кастомного менеджера
```

Теперь можно использовать:
```python
active_objects = MyModel.objects.active()
```

---

## 10. Транзакции

Работа с транзакциями:
```python
from django.db import transaction

with transaction.atomic():
    obj1 = MyModel.objects.create(field1='value1')
    obj2 = AnotherModel.objects.create(field2='value2')
```
[к содержанию](#содержание)
