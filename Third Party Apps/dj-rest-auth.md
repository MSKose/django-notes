# dj-rest-auth

- dj-rest-auth is a drop-in API endpoints for handling authentication securely in Django Rest Framework. Works especially well with SPAs (e.g React, Vue, Angular), and Mobile applications. See more [here](https://github.com/iMerica/dj-rest-auth)
- First, let's start with the installation:

```python
pip install dj-rest-auth
```

- Add `dj_rest_auth` app to INSTALLED_APPS in your django settings.py:

```python
INSTALLED_APPS = (
    ...,
    'rest_framework',
    'rest_framework.authtoken',
    ...,
    'dj_rest_auth'
)
```

- Add URL patterns:

```python
urlpatterns = [
    path('your-endpoint-here/', include('dj_rest_auth.urls')),
]
```

- Some endpoints this urls give us access to are the following. Also, say you have your dj_rest_auth urls defined in an app called users. Hence, the baseurl before dj_rest_auth is `localhost:8000/users/`;
    - users`/auth/login/`
    - users`/auth/logout/` —> Calls Django logout method and delete the Token object assigned to the current User object.
    - users`/user/` —> Reads and updates UserModel fields Accepts GET, PUT, PATCH methods.
    - users`/password/change/` —> Calls Django Auth SetPasswordForm save method.
    - users`/password/reset/` —> Calls Django Auth SetPasswordForm save method.
    - users`/password/change/confirm/` —> Password reset e-mail link is confirmed, therefore this resets the user's password.
    - users`/register/`
- Migrate your database

```python
python manage.py migrate
```

- dj-rest-auth comes with optional registration kinds like [allauth](https://dj-rest-auth.readthedocs.io/en/latest/installation.html#registration-optional). But if you'd like to write your own registration you can do that too
- However, apart from registration, `dj-rest-auth` come with some default [endpoints](https://dj-rest-auth.readthedocs.io/en/latest/api_endpoints.html) like `/login/`, `/logout/` and `/reset/`
- In the default rest-framework auth, logout didn't automatically delete our token. But with dj-rest-auth, it comes out of the box
- Want hands-on experience with dj-rest-auth? Try their demo project with back-end and front-end [here](https://dj-rest-auth.readthedocs.io/en/latest/demo.html)

## Overriding the return value

- First, let's see what our default return value for the `/auth/login/` endpoint is;

```python
{
    "key": "1385eb85cbb7fbc63bajdyf2378d114afb14e60be67"
}
```

- With the token above we can access this user's details by sending a `GET` request to `/auth/user/` after having logged in. We'll then get the following;

```python
{
    "pk": 5,
    "username": "TestUser",
    "email": "testuser@test.com",
    "first_name": "Test",
    "last_name": "User"
}
```

- However, say we do not want to send a GET request after logging in, just to access the username. We then have to override the return value for logging in. For instance, let's send token (which is included by default), username and email upon logging in. There's two things we have to do for this to happen. First, add the following to your serializer;

```python
# serializers.py
from dj_rest_auth.serializers import TokenSerializer

class UserSerializer(serializers.ModelSerializer): # adding a serializer to get username and email
    class Meta:
        model = User
        fields = (
            'username',
            'email'
        )

class CustomTokenSerializer(TokenSerializer): # we will be adding username and email thus the UserSerializer
    user = UserSerializer(read_only=True)

    class Meta(TokenSerializer.Meta):
        fields = (
            'key',
            'user'
        )

# Therefore, we're overriding the TokenSerializer where our return value for login is defined originally
# And on the Meta part we inheriting TokenSerializer.Meta since we do not want to define the Meta model 
# again, but if we had left that out we could very well have imported the model in the source code and 
# write the model again. Therfore, there's nothing fancy with the Meta inheritence
```

- Adding these to our serializers won't be enough since the code will still look into the source code and read TokenSerializer - which only returns the token by default. The second step is defining where our Django code should be looking for our return value for our login POST requests. Therefore, we have to add the following to our settings (as is stated in the [docs](https://dj-rest-auth.readthedocs.io/en/latest/configuration.html));

```python
# settings.py (or base.url if you have a seperated settings configured)

REST_AUTH_SERIALIZERS = {
    'TOKEN_SERIALIZER': 'users.serializers.CustomTokenSerializer',
}

# Here, we simply write the path to where we wrote our serializers and since we wrote our serializers
# in the users app, it is users.serializers.CustomTokenSerializer for our case
```