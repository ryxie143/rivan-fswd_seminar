## BACKEND INSTALLATION/SETUP

## Task 1: Install Python

```python
python --version
```

---

## Task 2: Virtual Environment Installation

```python
python -m venv env
```
to activate:
```python
env\Scripts\activate
```

---

## Task 3: Make a new file and name it `requirements.txt`:

```
asgiref
Django
django-cors-headers
djangorestframework
djangorestframework-simplejwt
PyJWT
pytz
sqlparse
psycopg2-binary
python-dotenv
```

In the terminal, paste this:

```bash
pip install -r requirements.txt
```

---

## Task 4: Creating a Django Project

```bash
django-admin startproject backend
cd backend
django-admin startapp api
```

---

## Task 5: Configure `settings.py` in the `backend` folder:

```python
from datetime import timedelta
from dotenv import load_dotenv
import os

load_dotenv()
```

In ALLOWED_HOST paste this
```python
ALLOWED_HOSTS = ["*"]

REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": (
        "rest_framework_simplejwt.authentication.JWTAuthentication",
    ),
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated",
    ],
}

SIMPLE_JWT = {
    "ACCESS_TOKEN_LIFETIME": timedelta(minutes=30),
    "REFRESH_TOKEN_LIFETIME": timedelta(days=1),
}

```

In `INSTALLED_APPS`, add:

```python
"api",
"rest_framework",
"corsheaders",
```

In `MIDDLEWARE`, add:

```python
"corsheaders.middleware.CorsMiddleware",
```

At the end of the script, add:

```python
CORS_ALLOW_ALL_ORIGINS = True
CORS_ALLOWS_CREDENTIALS = True
```

Drag the requirements.txt inside the backend folder

---

## Task 6: Understanding JWT Tokens 

Make a new file inside the `api` folder and name it `serializers.py`, and paste this:

```python
from django.contrib.auth.models import User
from rest_framework import serializers

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ["id", "username", "password"]
        extra_kwargs = {"password": {"write_only": True}}

    def create(self, validated_data):
        user = User.objects.create_user(**validated_data)
        return user
```

---

## Task 7: Configure `views.py`:

```python
from django.shortcuts import render
from django.contrib.auth.models import User
from rest_framework import generics
from .serializers import UserSerializer
from rest_framework.permissions import IsAuthenticated, AllowAny

class CreateUserView(generics.CreateAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = [AllowAny]
```

---

## Task 8: Configure `backend/urls.py`:

```python
from django.contrib import admin
from django.urls import path,include
from api.views import CreateUserView
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView

urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/user/register/", CreateUserView.as_view(), name="register"),
    path("api/token/", TokenObtainPairView.as_view(), name="get_token"),
    path("api/token/refresh/", TokenRefreshView.as_view(), name="refresh"),
    path("api-auth/", include("rest_framework.urls")),
]
```

---

## Task 9: Run your server:

```bash
python manage.py makemigrations
python manage.py migrate
python manage.py runserver
```

---

## Task 10: Navigate to Your URL

- `/api/user/register`
- `/api/token`
- `/api/token/refresh`

---

## Task 11: Configure `api/models.py`:

```python
from django.db import models
from django.contrib.auth.models import User


class Note(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name="notes")

    def __str__(self):
        return self.title
```

---

## Task 12: Insert these in `serializers.py`:

```python
from .models import Note

class NoteSerializer(serializers.ModelSerializer):
    class Meta:
        model = Note
        fields = ["id", "title", "content", "created_at", "author"]
        extra_kwargs = {"author": {"read_only": True}}
```

---

## Task 13: Insert these in `views.py`. 

Add `, NoteSerializer` beside `UserSerializer`:

```python
from .models import Note

class NoteListCreate(generics.ListCreateAPIView):
    serializer_class = NoteSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        user = self.request.user
        return Note.objects.filter(author=user)

    def perform_create(self, serializer):
        if serializer.is_valid():
            serializer.save(author=self.request.user)
        else:
            print(serializer.errors)

class NoteDelete(generics.DestroyAPIView):
    serializer_class = NoteSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        user = self.request.user
        return Note.objects.filter(author=user)
```

---

## Task 14: Make a new file inside the `api` folder and name it `urls.py`:

```python
from django.urls import path
from . import views

urlpatterns = [
    path("notes/", views.NoteListCreate.as_view(), name="note-list"),
    path("notes/delete/<int:pk>/", views.NoteDelete.as_view(), name="delete-note"),
]
```

---

## Task 15: Insert this in `backend/urls.py`:

```python
path("api/", include("api.urls")),
```

After this, do **Task 9** again.

## Task 16: Navigate in the url in your browser

- `/api/token/refresh`
- `/api/token`
- `api/notes`
