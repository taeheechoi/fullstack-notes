Tutorial from https://medium.com/django-rest/lets-build-a-basic-Test-review-backend-with-drf-part-1-652dd9b95485

## Django REST framework
### Install dependencies
```
$ pip install django
$ pip install djangorestframework
$ pip install markdown
```

#### Start project and app
```
$ django-admin startproject medium
$ python manage.py startapp reviews
```

### Database
settings.py
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

### Migration
```
$ python manage.py migrate
```

### Model
---- Read multiple times on Medium ----
Many-to-one relationships : (models.ForeignKey)
Many-to-many relationships : (models.ManyToManyField)

```
from django.db import models
from django.contrib.auth.models import User


class Example(models.Model):
    name = models.CharField(max_length=255)
    url = models.TextField()
    category = models.ManyToManyField(Category, related_name='examples')
    Test = models.ForeignKey(Test, on_delete=models.CASCADE, related_name='examples', related_query_name='example')
    price = models.DecimalField(max_digits=9, decimal_places=2)
    created = models.DateField(auto_now_add=True)
    updated = models.DateField(auto_now=True)

    class Meta:
        ordering = ['-created']

    def __str__(self):
        return self.name
```
related_name: Test_instance.examples.all(): Example instances that are related to Test
related_query_name: Test.objects.filter(example=xxxxx): example as a lookup parameter in a queryset

### Add an app
```
INSTALLED_APPS = [
    ...,
    'myapp' or 'myapp.apps.MyAppsConfig',
]
```

### Make migrations and migrate
```
$ python manage.py makemigrations # some changes have been made on models
$ python manage.py migrate # synchronizing the changes between model and schema in database
```

### Superuser
```
$ python manage.py createsuperuser
```

### Run server
```
$ python manage.py runserver
```

### Add models to admin site
```
from django.contrib import admin
from .models import Example, MyExample
from django.contrib.auth.models import Group

@admin.register(MyExample)
class TestAdmin(admin.ModelAdmin):
    list_display = ('pk', 'name', 'content')
    list_filter = ('category', )

admin.site.register(Example1)

admin.siate.unregister(Group)

admin.site.site_header = "Test Admin"
```

### serializer
```
INSTALLED_APPS = [
    ...
    'rest_framework',
]
```
Serializer querysets and model instances --> native Python data types --> JSON
Deserializer JSON --> querysets

```
from .models import Test
from rest_framework import serializers

class TestSerializer(serializers.ModelSerializer):

    class Meta:
        model = Test
        fields = ['pk', 'name', 'category'] # fields = '__all__' 
        # exclude = ['category']
        # read_only_fields = ['name'] or extra_kwargs = {'name': {'read_only': True}}

    def validate_title(self, value): # field level validation
        if 'django' not in value.lower():
            raise serializers.ValidationError("error message")
        
        return value

    def validate(self, data): # object level validation
        if data['start'] > data['finish']:
            raise serializers.ValidationError("finish must occur after start")
        return data

    def create(self, validated_data): # overriding serializer method
        return Comment.objects.create(**validated_data)

    def update(self, instance, validated_data): # overriding serializer method
        instance.email = validated_data.get('email', instance.email)
        instance.title = validated_data.get('content', instance.title)
        instance.save()
        return instance
    

```
With Django Rest Framework the validation is performed entirely on the serializer class

read Including extra context, Nested relationships on article

nested relations
```
from .models import Test, MyTest, User
from rest_framework import serializers
from django.utils.timezone import now

class TestSerializer(serializers.ModelSerializer):
    class Meta:
        model = Test
        fields = ['pk', 'name']

class MyTestSerializer(serialziers.ModelSerializer):
    test = TestSerializer(many=True, context={'request': request}) # to represent a to-many relationship, including extra context 

    class Meta:
        model = MyTest
        fields = ['pk', 'name', 'category']

class UserSerializer(serializers.ModelSerializer):
    days_since_joined = serializers.SerializerMethodField()

    class Meta:
        model = User

    def get_days_since_joine(self, obj):
        return (now() - obj.date_joined).days

```

### views, function based views, class based views(generic views, viewsets)
ViewSets = Generic Views + List of objects + Detail of one object, no config for urls, routers generates urls
Generic View Set = no GET, POST, PUT. but get_object and get_queryset included
ReadOnlyModelViewSet = GET, list(), retrive()
ModelViewSet = create(), retrieve(), update(), partial_update(), destroy(), list()

```
from rest_framework.viewsets import ReadOnlyModelViewSet
from .serializers import TestSerializer
from .models import Test


class TestViewSet(ReadOnlyModelViewSet):

    serializer_class = TestSerializer
    queryset = Test.objects.all()

    @action(detail=False)
    def get_list(self, request):
        pass

    @action(detail=True)
    def get_test(self, request, pk=None):
        pass

    @action(detail=True, methods=['post', 'delete'])
    def delete_test(self, request, pk=None):
        pass


```

### router
```
from django.contrib import admin
from django.conf.urls import url, include
from reviews.views import TestViewSet
from rest_framework.routers import DefaultRouter

router = DefaultRouter()
router.register(r'test', TestViewSet, basename='Test')

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^', include(router.urls)),
]
```

### Dynamic nested serialization
Read again
```
pip install drf-flex-fields
```
