# DRF ��������

## 1. ����ȯ�� ����
### 1-1. ������Ʈ ���丮 ���� �� �̵�
```bash
mkdir drf-practice
cd drf-practice
```
### 1-2. ����ȯ�� ����
```bash
python -m venv venv
```
### 1-3. ����ȯ�� Ȱ��ȭ
```bash
source ./venv/bin/activate
```
### 1-4. �ʿ� ��� ��ġ
```bash
pip install django djangorestframework
```
## 2. ��� ������Ʈ �⺻ ����
### 2-1. ��� ������Ʈ ����
```bash
django-admin startproject project .
```
### 2-2. blog �� ����
```bash
python manage.py startapp blog
```
### 2-3. project/settings.py ����
```python
ALLOWED_HOSTS = ['*']

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'blog',
]
...
LANGUAGE_CODE = 'ko-kr'

TIME_ZONE = 'Asia/Seoul'
...
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.UserRateThrottle',
    ],
    'DEFAULT_THROTTLE_RATES': {
        'user': '1000/day',
    }
}
```

## 3. Model �ۼ�
### 3-1. blog/models.py
```python
from django.db import models
from django.conf import settings

class Post(models.Model):
    author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title = models.CharField(max_length=100)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
class Notice(models.Model):
    author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title = models.CharField(max_length=100)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

### 3-2. �� ������� ����
```bash
python manage.py makemigrations
python manage.py migrate
```

## 4. URL ����
### 4-1. project/urls.py
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api-auth/', include('rest_framework.urls')),
    path('', include('blog.urls')),
]
```

### 4-2. blog/urls.py
```python
from django.urls import include, path
from rest_framework.routers import DefaultRouter
from . import views

router = DefaultRouter()
router.register('blog', views.PostViewSet)
router.register('notice', views.NoticeViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```

## 5. Views ����
### 5-1. blog/serializers.py
```python
from rest_framework.serializers import ModelSerializer
from .models import Post, Notice

class PostSerializer(ModelSerializer):
    class Meta:
        model = Post
        fields = '__all__'
        
class NoticeSerializer(ModelSerializer):
    class Meta:
        model = Notice
        fields = '__all__'
```

### 5-2. blog/permissions.py
```python
from rest_framework import permissions


class IsAuthorOrReadOnly(permissions.BasePermission):
    def has_permission(self, request, view):
        if request.method in permissions.SAFE_METHODS:
            return True
        return request.user.is_authenticated
    
    def has_object_permission(self, request, view, obj):
        if request.method in permissions.SAFE_METHODS:
            return True
        return obj.author == request.user
```

### 5-3. blog/views.py
```python
from rest_framework.viewsets import ModelViewSet
from .models import Post, Notice
from .serializers import PostSerializer, NoticeSerializer
from rest_framework.permissions import IsAuthenticated
from .permissions import IsAuthorOrReadOnly

class PostViewSet(ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    permission_classes = [IsAuthenticated, IsAuthorOrReadOnly]
    
class NoticeViewSet(ModelViewSet):
    queryset = Notice.objects.all()
    serializer_class = NoticeSerializer
    permission_classes = [IsAuthorOrReadOnly]
```

## 6. �׽�Ʈ�� �ϱ� ���� ���۾�
### 6-1. ������ ���� ����
```bash
python manage.py createsuperuser
```

### 6-2. ���� ����
```bash
python manage.py runserver
```

### 6-3. ���ο� ���� ���� (�Ϲݰ���)
- http://127.0.0.1:8000/admin/auth/user/ �� ���� ����ڸ� �߰��Ѵ�.

## 7. Blog CRUD �׽�Ʈ
### 7-1. �� �ۼ� (Create)
#### 7-1-1. �α��� O
**Request**
- URL: http://127.0.0.1:8000/blog/
- Method: POST
- Auth�� Basic�� �α��� ������ �Է��Ѵ�. (Username, Password)
- Body
```json
{
 "title": "Testest",
 "content": "testestest",
 "author": 2
}
```

**Response**
```json
Status: 201 Created
Size: 144 Bytes
Time: 656 ms
{
  "id": 5,
  "author": 2,
  "title": "Testest",
  "content": "testestest",
  "created_at": "2023-11-20T15:36:36.645341",
  "updated_at": "2023-11-20T15:36:36.645374"
}
```

#### 7-1-2. �α��� X
**Request**
- ���� ������ Auth �׸� ����.

**Response**
```json
Status: 403 Forbidden
Size: 96 Bytes
Time: 6 ms
{
  "detail": "�ڰ� ����������(authentication credentials)�� �������� �ʾҽ��ϴ�."
}
```

### 7-2. �� ��ȸ (Read)
#### 7-2-1. ��Ϻ���, �α��� O
**Request**
- URL: http://127.0.0.1:8000/blog/
- Method: GET
- Auth�� Basic�� �α��� ������ �Է��Ѵ�. (Username, Password)

**Response**
```json
Status: 200 OK
Size: 676 Bytes
Time: 672 ms
[
  {
    "id": 1,
    "author": 1,
    "title": "1",
    "content": "11",
    "created_at": "2023-11-20T15:13:53.235948",
    "updated_at": "2023-11-20T15:13:53.236018"
  },
  {
    "id": 2,
    "author": 1,
    "title": "2",
    "content": "22",
    "created_at": "2023-11-20T15:13:59.574850",
    "updated_at": "2023-11-20T15:13:59.574884"
  },
  {
    "id": 3,
    "author": 1,
    "title": "3",
    "content": "33",
    "created_at": "2023-11-20T15:14:04.885616",
    "updated_at": "2023-11-20T15:14:04.885656"
  },
  {
    "id": 4,
    "author": 2,
    "title": "44up",
    "content": "444up",
    "created_at": "2023-11-20T15:27:52.078566",
    "updated_at": "2023-11-20T15:28:14.467518"
  },
  {
    "id": 5,
    "author": 2,
    "title": "Testest",
    "content": "testestest",
    "created_at": "2023-11-20T15:36:36.645341",
    "updated_at": "2023-11-20T15:36:36.645374"
  }
]
```

#### 7-2-2. ��Ϻ���, �α��� X
**Request**
- ���� ������ Auth �׸� ����.

**Response**
```json
Status: 403 Forbidden
Size: 96 Bytes
Time: 15 ms
{
  "detail": "�ڰ� ����������(authentication credentials)�� �������� �ʾҽ��ϴ�."
}
```

#### 7-2-3. �󼼺���, �α��� O
**Request**
- URL: http://127.0.0.1:8000/blog/1/
- Method: GET
- Auth�� Basic�� �α��� ������ �Է��Ѵ�. (Username, Password)

**Response**
```json
Status: 200 OK
Size: 130 Bytes
Time: 431 ms
{
  "id": 1,
  "title": "1",
  "content": "11",
  "created_at": "2023-11-20T15:13:53.235948",
  "updated_at": "2023-11-20T15:13:53.236018",
  "author": 1
}
```

#### 7-2-4. �󼼺���, �α��� X
**Request**
- ���� ������ Auth �׸� ����.

**Response**
```json
Status: 403 Forbidden
Size: 96 Bytes
Time: 7 ms
{
  "detail": "�ڰ� ����������(authentication credentials)�� �������� �ʾҽ��ϴ�."
}
```

### 7-3. �� ���� (Update)
#### 7-3-1. �� �ۼ����� ���
**Request**
- URL: http://127.0.0.1:8000/blog/5/
- Method: PATCH
- Auth�� Basic�� �� �� ������ �α��� ������ �Է��Ѵ�. (Username, Password)
- Body
```json
{
 "title": "Testupdate",
 "content": "testupdatete"
}
```

**Response**
```json
Status: 200 OK
Size: 149 Bytes
Time: 597 ms
{
  "id": 5,
  "author": 2,
  "title": "Testupdate",
  "content": "testupdatete",
  "created_at": "2023-11-20T15:36:36.645341",
  "updated_at": "2023-11-20T15:46:03.719022"
}
```

#### 7-3-2. �� �ۼ��ڰ� �ƴ� ���
**Request**
- URL: http://127.0.0.1:8000/blog/5/
- Method: PATCH
- Auth�� Basic�� �� �� ������ �ƴ� �ٸ� ������ �α��� ������ �Է��Ѵ�. (Username, Password)
- Body
```json
{
 "title": "Testupdate11",
 "content": "testupdatete11"
}
```

**Response**
```json
Status: 403 Forbidden
Size: 72 Bytes
Time: 656 ms
{
  "detail": "�� �۾��� ������ ����(permission)�� �����ϴ�."
}
```

### 7-4. �� ���� (Delete)
#### 7-4-1. �� �ۼ����� ���
**Request**
- URL: http://127.0.0.1:8000/blog/5/
- Method: DELETE
- Auth�� Basic�� �� �� ������ �α��� ������ �Է��Ѵ�. (Username, Password)

**Response**
```json
Status: 204 No Content
Size: 0 Bytes
Time: 598 ms
```

#### 7-4-2. �� �ۼ��ڰ� �ƴ� ���
**Request**
- URL: http://127.0.0.1:8000/blog/5/
- Method: DELETE
- Auth�� Basic�� �� �� ������ �ƴ� �ٸ� ������ �α��� ������ �Է��Ѵ�. (Username, Password)

**Response**
```json
Status: 403 Forbidden
Size: 72 Bytes
Time: 616 ms
{
  "detail": "�� �۾��� ������ ����(permission)�� �����ϴ�."
}
```

## 8. Notice CRUD �׽�Ʈ
### 8-1. �� �ۼ� (Create)
#### 8-1-1. �α��� O
**Request**
- URL: http://127.0.0.1:8000/notice/
- Method: POST
- Auth�� Basic�� �α��� ������ �Է��Ѵ�. (Username, Password)
- Body
```json
{
 "title": "noticetest",
 "content": "noticetestest",
 "author": 1
}
```

**Response**
```json
Status: 201 Created
Size: 150 Bytes
Time: 676 ms
{
  "id": 5,
  "title": "noticetest",
  "content": "noticetestest",
  "created_at": "2023-11-20T16:45:22.856486",
  "updated_at": "2023-11-20T16:45:22.856670",
  "author": 1
}
```

#### 8-1-2. �α��� X
**Request**
- ���� ������ Auth �׸� ����.

**Response**
```json
Status: 403 Forbidden
Size: 96 Bytes
Time: 8 ms
{
  "detail": "�ڰ� ����������(authentication credentials)�� �������� �ʾҽ��ϴ�."
}
```

### 8-2. �� ��ȸ (Read)
#### 8-2-1. ��Ϻ���, �α��� O
**Request**
- URL: http://127.0.0.1:8000/notice/
- Method: GET
- Auth�� Basic�� �α��� ������ �Է��Ѵ�. (Username, Password)

**Response**
```json
Status: 200 OK
Size: 726 Bytes
Time: 900 ms
[
  {
    "id": 1,
    "title": "notice11",
    "content": "noticee11",
    "created_at": "2023-11-20T16:10:54.824960",
    "updated_at": "2023-11-20T16:10:54.825003",
    "author": 2
  },
  {
    "id": 2,
    "title": "notice2",
    "content": "noticee2",
    "created_at": "2023-11-20T16:31:42.051989",
    "updated_at": "2023-11-20T16:31:42.052083",
    "author": 2
  },
  {
    "id": 3,
    "title": "notice3",
    "content": "noticee3",
    "created_at": "2023-11-20T16:33:04.215799",
    "updated_at": "2023-11-20T16:33:04.215822",
    "author": 2
  },
  {
    "id": 4,
    "title": "notice3",
    "content": "noticee3",
    "created_at": "2023-11-20T16:35:25.512174",
    "updated_at": "2023-11-20T16:35:25.512259",
    "author": 2
  },
  {
    "id": 5,
    "title": "noticetest",
    "content": "noticetestest",
    "created_at": "2023-11-20T16:45:22.856486",
    "updated_at": "2023-11-20T16:45:22.856670",
    "author": 1
  }
]
```

#### 8-2-2. ��Ϻ���, �α��� X
**Request**
- ���� ������ Auth �׸� ����.

**Response**
```json
Status: 200 OK
Size: 726 Bytes
Time: 15 ms
[
  {
    "id": 1,
    "title": "notice11",
    "content": "noticee11",
    "created_at": "2023-11-20T16:10:54.824960",
    "updated_at": "2023-11-20T16:10:54.825003",
    "author": 2
  },
  {
    "id": 2,
    "title": "notice2",
    "content": "noticee2",
    "created_at": "2023-11-20T16:31:42.051989",
    "updated_at": "2023-11-20T16:31:42.052083",
    "author": 2
  },
  {
    "id": 3,
    "title": "notice3",
    "content": "noticee3",
    "created_at": "2023-11-20T16:33:04.215799",
    "updated_at": "2023-11-20T16:33:04.215822",
    "author": 2
  },
  {
    "id": 4,
    "title": "notice3",
    "content": "noticee3",
    "created_at": "2023-11-20T16:35:25.512174",
    "updated_at": "2023-11-20T16:35:25.512259",
    "author": 2
  },
  {
    "id": 5,
    "title": "noticetest",
    "content": "noticetestest",
    "created_at": "2023-11-20T16:45:22.856486",
    "updated_at": "2023-11-20T16:45:22.856670",
    "author": 1
  }
]
```

#### 8-2-3. �󼼺���, �α��� O
**Request**
- URL: http://127.0.0.1:8000/notice/1/
- Method: GET
- Auth�� Basic�� �α��� ������ �Է��Ѵ�. (Username, Password)

**Response**
```json
Status: 200 OK
Size: 144 Bytes
Time: 372 ms
{
  "id": 1,
  "title": "notice11",
  "content": "noticee11",
  "created_at": "2023-11-20T16:10:54.824960",
  "updated_at": "2023-11-20T16:10:54.825003",
  "author": 2
}
```

#### 8-2-4. �󼼺���, �α��� X
**Request**
- ���� ������ Auth �׸� ����.

**Response**
```json
Status: 200 OK
Size: 144 Bytes
Time: 14 ms
{
  "id": 1,
  "title": "notice11",
  "content": "noticee11",
  "created_at": "2023-11-20T16:10:54.824960",
  "updated_at": "2023-11-20T16:10:54.825003",
  "author": 2
}
```

### 8-3. �� ���� (Update)
#### 8-3-1. �� �ۼ����� ���
**Request**
- URL: http://127.0.0.1:8000/notice/5/
- Method: PATCH
- Auth�� Basic�� �� �� ������ �α��� ������ �Է��Ѵ�. (Username, Password)
- Body
```json
{
 "title": "Testupdate",
 "content": "testupdatete"
}
```

**Response**
```json
Status: 200 OK
Size: 149 Bytes
Time: 665 ms
{
  "id": 5,
  "title": "Testupdate",
  "content": "testupdatete",
  "created_at": "2023-11-20T16:45:22.856486",
  "updated_at": "2023-11-20T16:49:54.068525",
  "author": 1
}
```

#### 8-3-2. �� �ۼ��ڰ� �ƴ� ���
**Request**
- URL: http://127.0.0.1:8000/notice/5/
- Method: PATCH
- Auth�� Basic�� �� �� ������ �ƴ� �ٸ� ������ �α��� ������ �Է��Ѵ�. (Username, Password)
- Body
```json
{
 "title": "Testupdate11",
 "content": "testupdatete11"
}
```

**Response**
```json
Status: 403 Forbidden
Size: 72 Bytes
Time: 537 ms
{
  "detail": "�� �۾��� ������ ����(permission)�� �����ϴ�."
}
```

### 8-4. �� ���� (Delete)
#### 8-4-1. �� �ۼ����� ���
**Request**
- URL: http://127.0.0.1:8000/notice/5/
- Method: DELETE
- Auth�� Basic�� �� �� ������ �α��� ������ �Է��Ѵ�. (Username, Password)

**Response**
```json
Status: 204 No Content
Size: 0 Bytes
Time: 415 ms
```

#### 8-4-2. �� �ۼ��ڰ� �ƴ� ���
**Request**
- URL: http://127.0.0.1:8000/notice/5/
- Method: DELETE
- Auth�� Basic�� �� �� ������ �ƴ� �ٸ� ������ �α��� ������ �Է��Ѵ�. (Username, Password)

**Response**
```json
Status: 403 Forbidden
Size: 72 Bytes
Time: 420 ms
{
  "detail": "�� �۾��� ������ ����(permission)�� �����ϴ�."
}
```