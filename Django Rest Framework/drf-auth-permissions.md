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


- Because we set our BasicAuth globally, we have to provide what BasicAuth needs to be able to have a valid GET request. And BasicAuth requires one to send the username and password of the user to authorize (which is why BasicAuth is not very secure). Therefore, in Postman in the Authorization tab select Basic Auth and enter your username and password for which you have created with createsuperuser. Only after entering your username and password there, will you be able to see the student list when you send a GET request, or be able to do a POST request for that matter.

<!-- ![drf_views_overview.png](./images/postman-basicAuth.png) -->
<p align="center">
  <img src="./images/postman-basicAuth.png" alt="drf_views_overview.png" height="300px"/>
</p>

- And if we were to use `IsAdminUser` for our views permission setting, as the name suggest, you'll not only be sending your username and password but also you'll have to be admin. Therefore, for the views like below;

```python
# views.py
from rest_framework.permissions import IsAdminUser

class StudentList(generics.ListCreateAPIView):
    serializer_class = StudentSerializer
    queryset = Student.objects.all()
    permission_classes = [IsAdminUser] # notice now we changed it to IsAdminUser
```

- For the views we have above, if we were to send a username and password for a user who's not admin we'd get an permission error like the one below;

```python
{
    "detail": "You do not have permission to perform this action."
}
```

- And now let's try `IsAuthenticatedOrReadOnly`;

```python
# views.py
from rest_framework.permissions import IsAdmIsAuthenticatedOrReadOnlyinUser

class StudentList(generics.ListCreateAPIView):
    serializer_class = StudentSerializer
    queryset = Student.objects.all()
    permission_classes = [IsAuthenticatedOrReadOnly] # notice now we changed it to IsAuthenticatedOrReadOnly
```

- For the view function above, we'll be able to see our student list for our GET requests even if we are not logged in. However, it would not be possible to perform a POST request before getting authorized by providing with username and password. The following would be the error for trying to do a POST request without authorization;

```python
{
    "detail": "Authentication credentials were not provided."
}
```


- We have stated earlier that BasicAuth is not secure. Let's see why. Head over to your Header tab in Postman and see the Authorization key-value pair. And in the value part after the String "Basic " comes an encoded string, paste that into a base64 decoder in internet and you'll get your username and password returned in username:password format.

<!-- ![Screenshot 2022-10-02 at 12.00.47.png](./images/base64-decoding-img.png)

![Screenshot 2022-10-02 at 12.01.15.png](./images/basicauth-img.png) -->

<p align="center">
  <img src="./images/base64-decoding-img.png" alt="drf_views_overview.png" height="300px"/>
  <img src="./images/basicauth-img.png" alt="drf_views_overview.png" height="300px"/>
</p>


- Having seen how detrimental using BasicAuth might be, let's now see the more secure way to dealing with Authorization: **[TokenAuthentication](https://www.django-rest-framework.org/api-guide/authentication/#tokenauthentication)**;

```python
# settings.py
INSTALLED_APPS = [
    ...
    'rest_framework.authtoken'
]

# First things first, we'll need to 'rest_framework.authtoken' to our INSTALLED_APPS
# Also you might need to apply migrations with python manage.py migrate after this
```

- Also, change your REST_FRAMEWORK settings now to the following;

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ]
}
```

- Now that we have added TokenAuthentication, we'll see a new tab Tokens in our Admin panel from which we can generate tokens for users. For the sake of simplicity let's change our views permission to `IsAuthenticated`;

```python
# views.py
class StudentList(generics.ListCreateAPIView):
    serializer_class = StudentSerializer
    queryset = Student.objects.all()
    permission_classes = [IsAuthenticated]
```



- Now head back to Postman and change the auth type in Tab in `Authorization` to "`No Auth`" (was previously "`Basic Auth`"). And add a new key-value pair in `Headers` tab. Set "`Authorization`" as key and "`Token <yourTokenHere>`" as value. You can get your token key from the Admin Panel by creating one for each user. After you have ticked the key-value pair row you have added, you'll be able to do GET request. And now go ahead and unselect that key-value pair and you'll get an "Authentication credentials were not provided.” error.
<!-- ![Screenshot 2022-10-02 at 12.20.28 1.png](./images/adding-token-postman-img.png) -->

<p align="center">
  <img src="./images/adding-token-postman-img.png" alt="drf_views_overview.png" height="300px"/>
</p>

- It's all good, but this is feasible only with Postman and Admin Panel and is therefore is not practical since we want to automize the process. There are some ways to go about it. You can see some ways like using signals or using the `obtain_auth_token` in drf official [docs](https://www.django-rest-framework.org/api-guide/authentication/#tokenauthentication).
- Let's see how `obtain_auth_token` works. For that, let's first create a new app called user_api for login and register instances;

```python
python manage.py startapp user_api

# don't forget to add 'user_api' to INSTALLED_APPS in settings.py
```

- In the project's urls.py add the following;

```python
from django.urls import path, include

urlpatterns = [
    ...
    path('user/', include('user_api.urls')),
]
```

- And in the user_api urls.py add the following;

```python
from django.urls import path
from rest_framework.authtoken.views import obtain_auth_token

urlpatterns = [
    path('login/', obtain_auth_token, name="login"),
]

# in a nutshell, obtain_auth_token expects a username and password (as we'll see it in practice below) and sends the token associated with that user
```

- Now let's head to Postman and untick the Authorization we have added previously. And in the Body tab select row and add your username and password in a JSON format and send a POST request to the endpoint user/login/

```python
# therefore POSTing this;
{
    "username": "Mustafa", # your username and password should match
    "password": "pass321"
}

# we will get this;
{
    "token": "4e49a6aab4ba52hsd63f20b58795b12jhgqd72395"
}

# In Front-End terms what we did here is simply trying to log in with a username and password. And upon entering
# a valid username and password we recieved the token we will be using to validate that the user
```

- The token we received is the same token we have used previously, which we were able to get from Admin Panel. The only difference is now we can receive the token from Front-End too. And now try to send GET and POST requests in the endpoint api/student/ after adding the token to Headers, you'll be able to successfully send GET and POST requests. Therefore, what a Front-End does in this process is they add the token to Headers in their, say, axios method and be able to have the user logged in as long as the token in not missing in their request JSON.
- This was a login process only, if want to register the user from the front-end we probably will need a different endpoint. Since registration requires validation check we need to add a serializer;

```python
# serializers.py
from rest_framework import serializers
from django.contrib.auth.models import User
from rest_framework.validators import UniqueValidator
from django.contrib.auth.password_validation import validate_password

class RegistrationSerializer(serializers.ModelSerializer):
    # since email field is not by default added we are creating one
    email = serializers.EmailField(
            required=True,
            validators=[UniqueValidator(queryset=User.objects.all())]
            )

    password = serializers.CharField(write_only=True, required=True, validators=[validate_password]) # validators validates the password for the required standarts
    password2 = serializers.CharField(write_only=True, required=True)

    class Meta:
        model = User
        fields = ('username', 'password', 'password2', 'email', 'first_name', 'last_name')
        extra_kwargs = {
            'first_name': {'required': True},
            'last_name': {'required': True}
        }

    def validate(self, attrs): # validating if password and password2 matches 
        if attrs['password'] != attrs['password2']:
            raise serializers.ValidationError({"password": "Password fields didn't match."})

        return attrs

    def create(self, validated_data): # we are creating a user but we are not sending the password here since password is hashed
        user = User.objects.create(
            username=validated_data['username'],
            email=validated_data['email'],
            first_name=validated_data['first_name'],
            last_name=validated_data['last_name']
        )

        
        user.set_password(validated_data['password']) # we are creating the password this way since it's hashed
        user.save()

        return user

# now this serializer might look wordy, but it is what it is. Besides, you'll probably just copy 
# and paste this serializer whenever you need a registration serializer
```

```python
# views.py
from django.shortcuts import render
from .serializers import RegistrationSerializer
from rest_framework import generics
from django.contrib.auth.models import User

class RegisterView(generics.CreateAPIView):
    queryset = User.objects.all()
    serializer_class = RegistrationSerializer
```

- Having added our serizlizers and views, now let's add the view to our urls;

```python
# urls.py
from django.urls import path
from rest_framework.authtoken.views import obtain_auth_token
from .views import RegisterView

urlpatterns = [
    path('login/', obtain_auth_token, name="login"),
    path('register/', RegisterView.as_view(), name='register'),
]
```

- Now let's get back to Postman and test the endpoint user/register/. Choose the POST method and in the Body tab in row, this time send the fields we have had required in our serializer;

```python
{
    "username": "TestUser",
    "password": "pass321",
    "password2" : "pass321",
    "email" : "test@test.com",
    "first_name" : "Test",
    "last_name" : "User"
}

# after POSTing this, in your Admin Panel check the user TestUser being added
```

- But, by default, after adding the user TestUser we only get the following data being sent;

```python
{
    "username": "TestUser",
    "email": "test@test.com",
    "first_name": "Test",
    "last_name": "User"
}

# Therefore, we don't get the user's token being sent to us by default. We have to 
# add some more lines into our views overwriting the create method
```