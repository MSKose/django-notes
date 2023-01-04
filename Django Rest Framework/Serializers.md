# serializers

- Serializers in Django REST Framework are **responsible for converting objects into data types understandable by javascript and front-end frameworks**. Serializers also provide **deserialization**, allowing parsed data to be converted back into complex types, after first validating the incoming data.

💡 There are two ways to go about utilising serializers in DRF. First, the more verbose way, is to use `serializers.Serializer` and the second, to use the `serializers.ModelSerializer`. You might have noticed, this was the case with defining Forms too. Well, yes, serializers do resemble forms.


- Let’s start with the first method. Remember, defining a serializer looks very similar to defining a form:

```python
# serializers.py
from rest_framework import serializers

class CommentSerializer(serializers.Serializer):
    email = serializers.EmailField()
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
    number = serializers.IntegerField(required=False) # we had to write required=False since this field was null=True on our models.py

    def create(self, validated_data):
        return Student.objects.create(**validated_data)

    def update(self, instance, validated_data):
        instance.first_name = validated_data.get('first_name', instance.first_name) 
        instance.last_name = validated_data.get('last_name', instance.last_name)
        instance.number = validated_data.get('number', instance.number)
        instance.save()
        return instance

# Therefore, if you choose to write your code with serializers.Serializer you have to define create and update methods too
# We are sending instance.first_name as second argument in update, which means that if first_name is not changed, send the 
# current one. Same goes for last_name and number
```

- The second method to declare serializer is a lot less verbose. Thus, easy to read:

```python
# serializers.py
from rest_framework import serializers
from .models import Student

class StudentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Student
        fields = ["id", "first_name", "last_name", "number"]
        # exclude = ['number']

# Therefore, if you choose to write your code with serializers.ModelSerializer you don't have to define create and update methods
```

- We can even combine two fields and make a new one to pass into serializer fields:

```python
# serializers.py
class StudentSerializer(serializers.ModelSerializer):
    full_name = serializers.SerializerMethodField()

    def get_full_name(self, obj): # the syntax for defining added fields is get_<new_field_name>
        return f'{obj.first_name} {obj.last_name}'

    class Meta:
        model = Student
        fields = ["id", "full_name", "first_name", "last_name", "number"]

# notice how the field full_name does not exists on our Model but we combined two existing fields and created a new one
# do not forget to pass the "full_name" to fields in Meta
# other thing to note is that SerializerMethodField is a read-only field 
```

## Relational fields

- Say our model is as below, notice now Student class has a Many-to-One relationship with Path class:

```python
# models.py
from django.db import models

class Path(models.Model):
    path_name = models.CharField(max_length=50)

    def __str__(self):
        return f"{self.path_name}"

class Student(models.Model):
    path = models.ForeignKey(Path, related_name='students', on_delete=models.CASCADE)
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
    number = models.IntegerField(blank=True, null=True)

    def __str__(self):
        return f"{self.last_name} {self.first_name}"
```

```python
# serializers.py
from rest_framework import serializers
from .models import Student, Path

class PathSerializer(serializers.ModelSerializer):
    class Meta:
        model = Path
        fields = ["id", "path_name"]

class StudentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Student
        fields = ["id", "path", "first_name", "last_name", "number"]

# now we will also see path id when our StudentSerializer is utilised in views.py. Notice how I said path
# id and not path names. Say we had added the paths, FS and DS respectively. With this serializers.py we'd
# get a key-value pair of either "path": 1 or "path": 2. To get use of the dunder str method we had defined
# in our model, we'll have to use the StringRelatedField() method. See below:
```

```python
# serializers.py 
from rest_framework import serializers
from .models import Student, Path

class PathSerializer(serializers.ModelSerializer):
    students = serializers.StringRelatedField(many=True)
    class Meta:
        model = Path
        fields = ["id", "path_name"]

class StudentSerializer(serializers.ModelSerializer):
    path = serializers.StringRelatedField()
    class Meta:
        model = Student
        fields = ["id", "path", "first_name", "last_name", "number"]

# now that we have defined the path with StringRelatedField, we'll see FS or DS as keys on our respective objects
# StudentSerializer.
# In our PathSerializer we defined a StringRelatedField too, but this time with an additional
# argument, many=True. This is because several student can share the same path
```

- All good until we want to change the path with a PUT (or PATCH) on the browser. Say, for a student json object we had a path name "FS" and we wanted to change it to "DS", we cannot just pass "DS".  We'd get "*Incorrect type. Expected pk value, received str.*" error. This happens because `StringRelatedField()` is read-only. Instead we need an additional `IntegerField()` defined and have our changes with this key-value:

```python
# serializers.py 
class PathSerializer(serializers.ModelSerializer):
    students = serializers.StringRelatedField(many=True)
    class Meta:
        model = Path
        fields = ["id", "path_name"]

class StudentSerializer(serializers.ModelSerializer):
    path = serializers.StringRelatedField()
    path_id = serializers.IntegerField() # notice that this is an IntegerField

    class Meta:
        model = Student
        fields = ["id", "path", "path_id", "first_name", "last_name", "number"]

# having defined path_id and passed it as a field, now we can do updates on our json objects just fine. We'll 
# have to use the path_id to switch between paths. So say "FS" has id 1 and "DS" has 2. We should pass 2 if 
# it is 1, and vice versa, to switch between paths. 
```

<aside>
💡 An important note about path_id: the reason we used path_id to change paths with PUT and PATCH options is because that is how database has it. Open up your database for students and you'll indeed see that you have first_name, last_name, number, and path_id as table headers. Therefore, it really does make sense why we can only do updates with path_id and not path (which is read-only).

</aside>

- Also, another thing to note here is that if we hadn't define `path = serializers.StringRelatedField()` and `path_id = serializers.IntegerField()` in our serializers.py and passed "path" to Meta fields. We'd see key-value pairs of "path": 1's and "path": 2's because by default database recognizes our relevant path. But since, say our front-end could utilise a more human readable path name, we (re)defined path. But since this is a StringRelatedField, it is read-only and would let us only have GET requests and not POST or PUT. Therefore, having (re)defined path now we need to access path_id to be able to do updates on our instances. Therefore, we defined path_id alongside with path so that both our front-end and back-end would be happy!
- Say we wanted to see all the students registered to their respective paths in a json. How would we go about it? Filtering and for loops come to mind, right? Well, with DRF there is no need to do that. First we have to put the PathSerializer below the StudentSerializer since we want StudentSerializer to have defined already when we define PathSerializer because we will utilise the StudentSerializer in PathSerializer. Like so:

```python
class StudentSerializer(serializers.ModelSerializer):
    path=serializers.StringRelatedField()
    path_id=serializers.IntegerField()

    class Meta:
        model = Student
        fields = ["id","path_id","path","first_name", "last_name", "number"]

class PathSerializer(serializers.ModelSerializer):
    students = StudentSerializer(many=True) # many=True since we may have more than one student attached to a path

    class Meta:
        model = Path
        fields = ["id", "path_name", 'students']

# here, in PathSerializer "students" comes from related_name='students' on our model Student's path attribute. Had we given a different name 
# there, we would have used that name here. Therefore, related_name is exteremely important for these kinds of things. To see the related_name 
# see above codes where we had created our Model
```

- The following would be the output after defining our views and combining it with a url (we will be seeing more on views and url on the next chapter, let's focus on the serializer part for now):

```python
[
    {
        "id": 1,
        "path_name": "FS",
        "students": [
            {
                "id": 1,
                "path_id": 1,
                "path": "FS",
                "first_name": "Mustafa updated",
                "last_name": "Kose",
                "number": 123
            },
            {
                "id": 2,
                "path_id": 1,
                "path": "FS",
                "first_name": "Anthony",
                "last_name": "H",
                "number": 12345
            },
            {
                "id": 6,
                "path_id": 1,
                "path": "FS",
                "first_name": "Hakan",
                "last_name": "rain",
                "number": 123
            }
        ]
    },
    {
        "id": 2,
        "path_name": "DS",
        "students": [
            {
                "id": 3,
                "path_id": 2,
                "path": "DS",
                "first_name": "Henry",
                "last_name": "F",
                "number": 123456
            },
            {
                "id": 4,
                "path_id": 2,
                "path": "DS",
                "first_name": "Henry",
                "last_name": "F",
                "number": 12345
            }
        ]
    },
    {
        "id": 3,
        "path_name": "AWS",
        "students": [
            {
                "id": 5,
                "path_id": 3,
                "path": "AWS",
                "first_name": "Esad",
                "last_name": "A.",
                "number": 12345
            }
        ]
    }
]
```

- How cool is that? The only thing we had to do was to move the Path serializer class below Student serializer and to define a students variable matching with the related_name on our Student model and serializer.