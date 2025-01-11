# Django Validators

### django.core.validators:
валидаторы типа `MinLengthValidator` применяются указанием параметров поля (например, `MinLengthValidator` при указании min_length).
- `BaseValidator` — Базовый класс для следующих валидаторов: 
- `MaxValueValidator` — Проверяет, что значение не превышает максимум.  
- `MinValueValidator` — Проверяет, что значение не меньше минимума.  
- `MaxLengthValidator` — Проверяет, что длина строки не превышает максимум.  
- `MinLengthValidator` — Проверяет, что длина строки не меньше минимума.
- `StepValueValidator` - проверяет, что значение поля кратно заданному числу.

- `RegexValidator` — Проверяет соответствие строки регулярному выражению.  
- `EmailValidator` — Проверяет корректность email.  
- `URLValidator` — Проверяет корректность URL.
- `DomainNameValidator` - проверяет корректность доменного имени.

- `validate_email` — Проверяет корректность email (EmailValidator) автоматом вызывается в сериализаторах drf
- `validate_ipv4_address` — Проверяет корректность IPv4-адреса.  
- `validate_ipv6_address` — Проверяет корректность IPv6-адреса.  
- `validate_ipv46_address` — Проверяет корректность IPv4 или IPv6-адреса.  
- `validate_slug` — Проверяет, что строка состоит из букв, цифр, подчеркиваний и дефисов.  
- `validate_unicode_slug` — Проверяет корректность Unicode-slug.  
- `int_list_validator` — Проверяет, что строка — это список целых чисел. 

- `DecimalValidator` - Проверяет, что значение является десятичным числом
- `FileExtensionValidator` — Проверяет, что файл имеет допустимое расширение.
- `ProhibitNullCharactersValidator` — Проверяет отсутствие нулевых символов.

### django.contrib.auth.password_validation

- `CommonPasswordValidator` — Проверяет, что пароль не из списка часто используемых.  
- `MinimumLengthValidator` — Проверяет минимальную длину пароля.  
- `NumericPasswordValidator` — Проверяет, что пароль не состоит только из цифр.  
- `UserAttributeSimilarityValidator` — Проверяет, что пароль не похож на атрибуты пользователя.
- `validate_password` - применяет все валидаторы указанные в settings.AUTH_PASSWORD_VALIDATORS, не вызывается автоматически в DRF-сериализаторах(только в формах джанги).
- password_changed ??

### django.core.files.validators

- `FileExtensionValidator` — Проверяет расширение файла.  
- `validate_image_file_extension` — Проверяет расширение изображения.  
- `get_available_image_extensions` — Возвращает список доступных расширений изображений.

### django.utils.deconstruct

- `deconstructible` — Декоратор для сериализуемых валидаторов.

### django.contrib.postgres.validators

- `ArrayMinLengthValidator` — Проверяет минимальную длину массива.  
- `ArrayMaxLengthValidator` — Проверяет максимальную длину массива.

### django.contrib.gis.validators

- `validate_lonlat` — Проверяет корректность пары долготы и широты.

