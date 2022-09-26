# Function Based Views

- Function based views are, as you would have thought already, more verbose than class based views. We'll see about CBV later. Now let's see some codes from our project:

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
from .models import Student,Path
    
class StudentSerializer(serializers.ModelSerializer):
    path=serializers.StringRelatedField()
    path_id=serializers.IntegerField()

    class Meta:
        model = Student
        fields = ["id","path_id","path","first_name", "last_name", "number"]

class PathSerializer(serializers.ModelSerializer):
    students = StudentSerializer(many=True)

    class Meta:
        model = Path
        fields = ["id", "path_name", 'students']
```

- The `GET` method:

```python
# views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response
from .models import Student, Path
from .serializers import StudentSerializer, PathSerializer

@api_view(['GET']) # this decorator makes sure we get the respective method
def student_list(request):
    students = Student.objects.all() # we could also request filtered data with say filter(path=1)
    serializer = StudentSerializer(students, many=True) # many=True since we'll have more than one student on our db
    # print(serializer.data)
    return Response(serializer.data)

# urls.py
from django.urls import path, include
from .views import (
    student_list,
)

urlpatterns = [
    path('student_list/', student_list, name='student_list'),
]
```

- The `POST` method:

```python
from rest_framework import status

# views.py
@api_view(['POST'])
def student_create(request):
    print(request.data)
    serializer = StudentSerializer(data=request.data)
    if serializer.is_valid():
        serializer.save()
        data = {
            "message": f"Student {serializer.validated_data.get('first_name')} saved successfully!"
        }
        return Response(data, status=status.HTTP_201_CREATED) # status returns respective status code if successfull
    return Response(serializer.errors)

# urls.py
urlpatterns = [
    ...
    path('student_create/', student_create, name='student_create'),
]
```

- `GET` and `POST` in one view function:

```python
@api_view(['GET', 'POST'])
def student_api(request):
    if request.method == 'GET':
        students = Student.objects.all()
        serializer = StudentSerializer(students, many=True)
        return Response(serializer.data)
    elif request.method == 'POST':
        serializer = StudentSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            data = {
                "message": f"Student {serializer.validated_data.get('first_name')} saved successfully!"}
            return Response(data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

<aside>
💡 Also, keep in mind that best practice is to combine methods in one function and endpoint. Thus, writing the GET and POST in one function is actually the preferred way.

</aside>

- Getting individual elements with `GET`:

```python
from django.shortcuts import get_object_or_404 # this, under the hood is a try except block, where it returns 404 if data it not right

@api_view(['GET'])
def student_detail(request, pk): # because it's individual objects we're after, we're sending the pk too this time
    student = get_object_or_404(Student, pk=pk)
    serializer = StudentSerializer(student) # this time we didn't put many=True since we'll only get one object
    return Response(serializer.data, status=status.HTTP_200_OK)

# urls.py
urlpatterns = [
    ...
    path('student_detail/<int:pk>/', student_detail, name='student_detail'),
]
```

- The `PUT` method:

```python
@api_view(['PUT'])
def student_update(request, pk):
    student = get_object_or_404(Student, pk=pk)
    serializer = StudentSerializer(student, data=request.data)
    if serializer.is_valid():
        serializer.save()
        data = {
            "message": f"Student {student.last_name} updated successfully"
        }
        return Response(data, status=status.HTTP_200_OK)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

# urls.py
urlpatterns = [
    ...
    path('student_update/<int:pk>/', student_update, name='student_update'),
]
```

- If we hadn't sent every key-value pairs in PUT method, we'd get `"this field is required"` error for the key's we didn't update. Therefore, we may update only partially with the PATCH method:

```python
@api_view(['PATCH'])
def student_update_partial(request, pk):
    student = get_object_or_404(Student, pk=pk)
    serializer = StudentSerializer(student, data=request.data, partial=True) # partial=True part basically makes this a PATCH method
    if serializer.is_valid():
        serializer.save()
        data = {
            "message": f"Student {student.last_name} updated successfully"
        }
        return Response(data, status=status.HTTP_200_OK)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

# urls.py
urlpatterns = [
    ...
    path('student_update_partial/<int:pk>/', student_update_partial, name='student_update_partial'),
]
```

- The `DELETE` method:

```python
@api_view(['DELETE'])
def student_delete(request, pk):
    student = get_object_or_404(Student, pk=pk)
    student.delete()
    data = {
        "message": f"Student {student.last_name} deleted successfully..."
    }
    return Response(data, status=status.HTTP_200_OK)

# urls.py
urlpatterns = [
    ...
    path('student_delete/<int:pk>/', student_delete, name='student_delete'),
]
```

- `GET`, `PUT`, `DELETE`, and `PATCH` in one function:

```python
@api_view(['GET', 'PUT', 'DELETE', 'PATCH'])
def student_api_get_update_delete(request, pk):
    student = get_object_or_404(Student, pk=pk)
    if request.method == 'GET':
        serializer = StudentSerializer(student)
        return Response(serializer.data, status=status.HTTP_200_OK)
    elif request.method == 'PUT':
        serializer = StudentSerializer(student, data=request.data)
        if serializer.is_valid():
            serializer.save()
            data = {
                "message": f"Student {student.last_name} updated successfully"
            }
            return Response(data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    elif request.method == 'PATCH':
        serializer = StudentSerializer(
            student, data=request.data, partial=True)
        if serializer.is_valid():
            serializer.save()
            data = {
                "message": f"Student {student.last_name} updated successfully"
            }
            return Response(data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    elif request.method == 'DELETE':
        student.delete()
        data = {
            "message": f"Student {student.last_name} deleted successfully"
        }
        return Response(data)

# urls.py
urlpatterns = [
    ...
    path('student/<int:pk>/', student_api_get_update_delete, name="detail"),
]
```