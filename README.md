## Introduction
Some useful utils, BaseClass, decorator for drf development

## Installation

    pip install rachel 
    
... or clone the repo from GitHub and then:

    python setup.py install

## QuickStart

### Serializer
Inherit from ModelSerializer, add a `process_initial_data` method to process init data, so we can format data in serializer

Usage:

```python

from django.db import models
from rachel.serializers import Serializer

class User(models.Model):
    name = models.CharField('name',max_length=256,null=True, blank=True)


class UserSerializer(Serializer):
    
    class Meta:
        model = User
        fields = '__all__'
    
    def process_initial_data(self):
        if "name" not in self.initial_data and "user_name" in self.initial_data:
            self.initial_data['name'] = self.initial_data['user_name']

data = {"user_name": "rachel"}

s = UserSerializer(data=data)
s.is_valid(raise_exception=True)
user = s.save()
```


### NestedModelField

serialise to model instance According to `lookup_field` in model and `looup_field_name` in data

Usage:
```python

from rachel.serializers import Serializer
from rachel.fields import NestedModelField


class Email(models.Model):
    user = models.ForeignKey(User, on_delete=models.PROTECT)

class EmailSerializer(Serializer):
    username = models.CharField('email',max_length=150,unique=True)
    user = NestedModelField(model=User, lookup_field="id", lookup_field_name="user_id")
    class Meta:
        model = "user", "username"
data = {"user_id": 1, "email": "someone@email.com"}
ss = EmailSerializer(data)
ss.is_valid()
email = ss.save()
print(email, email.user)
# <Email someone@email.com>, <User 1>

```
### viewset && routers
!! important must discovery all viewset

Usage:
```python
from rachel.views import ViewSet 
from rachel.routers import router

class EmailViewSet(ViewSet):
    name = "email"
    path = "emails"
    model = Email
    serializer_class = EmailSerializer
    
class UserViewSet(ViewSet):
    name = "user"
    path = "users"
    model = User
    serializer_class = UserSeriliazer    
    filter_fields = "name","id"

    
router.custom_register(UserViewSet)
```

add urls in your project urls

```python
from rachel.routers import router
# must discovery all viewset
urlpatterns = [
    # url(r'^admin/', admin.site.urls),
    url(r'^v1/', include(router.urls)),   

```

Now, your project has the following four urls

- /v1/users/{user_pk}/ # allow http action `GET`, `Delete`, `Patch` 
- /v1/users/ # allow http action `Get`, `Post'
- /v1/users/{user_pk}/emails/{email_pk}/ # allow http action `GET`, `Delete`, `Patch` 
- /v1/users/{user_pk}/emails/ # allow http action `Get`, `Post'

### StateMachine

combined [transitions](https://github.com/pytransitions/transitions) with django model. 

### filter support

GET /v1/users/?name=ethan&id__gt=3

```python
query = Users.objects.all().filter(name="ethan").filter(id__gt=3)
```

### decorators  
#### class_method_retry
when specific error occurs, retry specific times after sleep second

#### viewset_url_params_require
define mandatory params, if not receive, raise `RequestQueryParamsMissing` Error