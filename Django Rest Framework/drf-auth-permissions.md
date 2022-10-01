# DRF-Auth-Permissions

- Permissions determine whether a request should be granted or denied access. There are layers into how you can define permissions. You can define it on settings and get global permissions apply, or you can set permissions on certain views specifically. Better yet, you can define permissions only for certain objects; say, for a student.
- Authorization checks who sends this request.
- You get some some built-in permission policies like; **[AllowAny](https://www.django-rest-framework.org/api-guide/permissions/#allowany)** (this is the default)**, [IsAuthenticated](https://www.django-rest-framework.org/api-guide/permissions/#isauthenticated), [IsAdminUser](https://www.django-rest-framework.org/api-guide/permissions/#isadminuser),** and **[IsAuthenticatedOrReadOnly](https://www.django-rest-framework.org/api-guide/permissions/#isauthenticatedorreadonly)**
- The built-in authorization policies are as follows: **[BasicAuthentication](https://www.django-rest-framework.org/api-guide/authentication/#basicauthentication)** (not so secure), **[TokenAuthentication](https://www.django-rest-framework.org/api-guide/authentication/#tokenauthentication), [SessionAuthentication](https://www.django-rest-framework.org/api-guide/authentication/#sessionauthentication), [RemoteUserAuthentication](https://www.django-rest-framework.org/api-guide/authentication/#remoteuserauthentication)**

<aside>
💡 Best practice for setting permission policy is to use your views; therefore, setting them locally. But, for authorization mostly global settings should be preferred.

</aside>

## Authorization

- Let’s set a basic authorization globally:

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.BasicAuthentication',
    ]
}
```

- Say we have a StudentList view and the corresponding endpoint for that view is /api/student/ and we want to add `IsAuthenticated` permission, we'd do that as following:

```python
# views.py
from rest_framework.permissions import IsAuthenticated

class StudentList(generics.ListCreateAPIView):
    serializer_class = StudentSerializer
    queryset = Student.objects.all()
    permission_classes = [IsAuthenticated] # "permission_classes" is what we use for permissions
```

- Before adding the above codes into my project, I'd get the students listed when I used the /api/student/ endpoint. But after adding them, I'll get the following error:

```python
{
    "detail": "Authentication credentials were not provided."
}
```

<!-- ![drf_views_overview.png](./images/postman-basicAuth.png) -->
<p align="center">
  <img src="./images/postman-basicAuth.png" alt="drf_views_overview.png" height="300px"/>
</p>
- Because we set BasicAuth globally we have to provide what BasicAuth needs to see the list. And BasicAuth requires you one to send the username and password of the user to authorize (which is why BasicAuth is not very secure). Therefore, in Postman in the Authorization tab select Basic Auth and enter your username and password for which you have created with createsuperuser. Only after entering your username and password there, will you be able to see the student list when you send a GET request, or be able to do a POST request for that matter.
-