# TDD ��������
������

## ����
1. ����ȯ�� ����
2. ������Ʈ ����
3. ������Ʈ �⺻ ����
4. Model �ۼ�
5. Urls, Views, Templates �ۼ�
6. Test �ڵ� �ۼ�
7. Test ����
8. Test ���

## ����ȯ�� ����
1. ������Ʈ ���͸� ����  
```
mkdir assignment
```

2. ����ȯ�� ����  
```
python -m venv venv
```

3. ����ȯ�� ����  
```
source venv/bin/activate
```

4. Django ��ġ  
```
pip install django
```

5. ������Ʈ ���͸��� �̵�  
```
cd assingment
```

## ������Ʈ ����
```
django-admin startproject assignment .
```

## ������Ʈ �⺻ ����
1. DB ���̱׷���Ʈ  
```
python manage.py migrate
```

2. Blog �� ����  
```
python manage.py startapp blog
```

3. assignment > settings.py ����  
```python
ALLOWED_HOSTS = ['*']

INSTALLED_APPS = [
    ...
    "blog",
]

TEMPLATES = [
    {
        ...
        "DIRS": [BASE_DIR/'templates'],
        ...
    },
]
```

4. ���� ���� Ȯ��
```
python manage.py runserver
```

## Model �ۼ�
1. blog > models.py
```python
from django.db import models

class Post(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateField(auto_now=True)
```
2. model ������� �ݿ�
```python
python manage.py makemigrations
python manage.py migrate
```
3. admin ������ ���
blog > admin.py
```python
from django.contrib import admin
from .models import Post

admin.site.register(Post)
```

## Urls, Views, Templates �ۼ�
1. assignment > urls.py
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path('', include('blog.urls')),
]
```

2. blog > urls.py
```python
from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    path('', views.index, name='index'),
    path('about/', views.about, name='about'),
    path('contact/', views.contact, name='contact'),
    path('blog/', views.blog_list, name='blog_list'),
    path('blog/<int:pk>', views.blog_detail, name='blog_detail'),
]
```

3. blog > views.py
```python
from django.db.models.query import QuerySet
from django.shortcuts import render
from django.views.generic import TemplateView, ListView, DetailView
from .models import Post
from django.db.models import Q

class IndexView(TemplateView):
    template_name = 'blog/index.html'

class AboutView(TemplateView):
    template_name = 'blog/about.html'

class ContactView(TemplateView):
    template_name = 'blog/contact.html'

class PostListView(ListView):
    model = Post
    context_object_name = 'posts'

    def get_queryset(self):
        queryset = super().get_queryset()

        q = self.request.GET.get('q', '')
        if q:
            queryset = Post.objects.filter(
                Q(title__icontains=q)
            )

        return queryset

class PostDetailView(DetailView):
    model = Post
    context_object_name = 'post'


index = IndexView.as_view()
about = AboutView.as_view()
contact = ContactView.as_view()
blog_list = PostListView.as_view()
blog_detail = PostDetailView.as_view()
```

4. Templates �ۼ�
- base.html
```html
<!DOCTYPE html>
<html lang="ko-KR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Main</title>

    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.1/dist/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.1/dist/js/bootstrap.bundle.min.js"></script>

</head>
<body>
    <!-- navbar -->
    <header>
        <nav class="navbar navbar-expand-lg navbar-light bg-light">
            <div class="container-fluid">
            <a class="navbar-brand" href="{% url 'blog:index' %}">TDD TEST</a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav">
                <li class="nav-item">
                    <a class="nav-link active" aria-current="page" href="{% url 'blog:index' %}">Home</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link" href="{% url 'blog:about' %}">About</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link" href="{% url 'blog:contact' %}">Contact</a>
                </li>
                </ul>
            </div>
            </div>
        </nav>
    </header>
    <!-- //navbar -->
    {% block contents %}
    {% endblock %}
    <!-- footer -->
    <footer class="footer mt-auto p-3 bg-white" style="bottom: 0;">
        <div class="d-flex container-fluid justify-content-between">
            <span class="text-muted fs-5">Copyright 2023. ������ all rights reserved.</span>
            
        </div>
    </footer>
		<!-- //footer -->
</body>
</html>
```
- blog > index.html
```html
{% extends 'base.html' %}
{% block contents %}
<main>
    <div>
        Main ������ �Դϴ�.
    </div>
</main>
{% endblock %}
```
- blog > about.html
```html
{% extends 'base.html' %}
{% block contents %}
<main>
    <div>
        About ������ �Դϴ�.
    </div>
</main>
{% endblock %}
```
- blog > contact.html
```html
{% extends 'base.html' %}
{% block contents %}
<main>
    <div>
        Contact ������ �Դϴ�.
    </div>
</main>
{% endblock %}
```

- blog > post_list.html
```html
{% extends 'base.html' %}
{% block contents %}
<main>
    <form method="get">
        <div>
            <div class="row g-2">
                <div class="col">
                    <input class="form-control" type="search" name="q">
                </div>
                <div class="col">
                    <button class="btn btn-primary" type="submit">�˻��ϱ�</button>
                </div>
            </div>
        </div>
    </form>
    {% if posts %}
        {% for post in posts %}
        <div>
            <section class='contents-section'>
                <h2 class='contents-heading'>{{ post.title }}</h2>
                <p class='contents-text'>{{ post.content }}</p>
                <p class='contents-updated'>{{ post.updated_at }}</p>
            </section>
        </div>
        {% endfor %}
    {% else %}
    <p>�Խù��� �������� �ʽ��ϴ�. ù��° �Խù��� ���ΰ��� �Ǽ���!</p>
    {% endif %}
</main>
{% endblock %}
```

- blog > post_detail.html
```html
{% extends 'base.html' %}
{% block contents %}
<main>
    <div>
        <section class='contents-section'>
            <h2 class='contents-heading'>{{ post.title }}</h2>
            <p class='contents-text'>{{ post.content }}</p>
            <p class='contents-updated'>{{ post.updated_at }}</p>
        </section>
    </div>
</main>
{% endblock %}
```

## Test �ڵ� �ۼ�
blog > tests.py
```python
from django.test import TestCase, Client
from bs4 import BeautifulSoup
from .models import Post

class Test(TestCase):
    def setUp(self):
        '''
        �׽�Ʈ�� ���� ���ο� Ŭ���̾�Ʈ ����
        '''

        self.client = Client()

    def test_connection(self):
        '''
        ���� ���� Ȯ�� �׽�Ʈ�� �����մϴ�
        '''

        print('---Main ������ ���� �׽�Ʈ---')
        response = self.client.get('/')
        self.assertEqual(response.status_code, 200)

        print('---About ������ ���� �׽�Ʈ---')
        response = self.client.get('/about/')
        self.assertEqual(response.status_code, 200)

        print('---Contact ������ ���� �׽�Ʈ---')
        response = self.client.get('/contact/')
        self.assertEqual(response.status_code, 200)

        print('---About ������ ���� �׽�Ʈ---')
        response = self.client.get('/blog/')
        self.assertEqual(response.status_code, 200)

    def test_template_inheritance(self):
        '''
        base.html�� ���� ���ø� ��� �׽�Ʈ�� �����մϴ�
        '''
        
        print('---Main ������ ��� �׽�Ʈ---')
        response = self.client.get('/')
        soup = BeautifulSoup(response.content, 'html.parser')
        self.assertTrue(soup.find('header'))
        self.assertTrue(soup.find('body'))
        self.assertTrue(soup.find('footer'))

        print('---About ������ ��� �׽�Ʈ---')
        response = self.client.get('/about/')
        soup = BeautifulSoup(response.content, 'html.parser')
        self.assertTrue(soup.find('header'))
        self.assertTrue(soup.find('body'))
        self.assertTrue(soup.find('footer'))

        print('---Contact ������ ��� �׽�Ʈ---')
        response = self.client.get('/contact/')
        soup = BeautifulSoup(response.content, 'html.parser')
        self.assertTrue(soup.find('header'))
        self.assertTrue(soup.find('body'))
        self.assertTrue(soup.find('footer'))

        print('---Blog ������ ��� �׽�Ʈ---')
        response = self.client.get('/blog/')
        soup = BeautifulSoup(response.content, 'html.parser')
        self.assertTrue(soup.find('header'))
        self.assertTrue(soup.find('body'))
        self.assertTrue(soup.find('footer'))

    def test_blog_list(self):
        '''
        �Խù� ��Ͽ� ���� ���� ��츦 �׽�Ʈ�մϴ�.
        '''

        print('---Blog ������ ����Ʈ�� �Խù��� ���� ��� �׽�Ʈ---')
        # Post �� 2�� �ֱ�
        post_001 = Post.objects.create(
            title= 'ù��° �Խù� �����Դϴ�.',
            content= 'ù��° �Խù� �����Դϴ�.',
        )
        post_002 = Post.objects.create(
            title= '�ι�° �Խù� �����Դϴ�.',
            content= '�ι�° �Խù� �����Դϴ�.',
        )
        response = self.client.get('/blog/')
        soup = BeautifulSoup(response.content, 'html.parser')

        # ���� �ش��ϴ� �±� h2�� 1������ ���� �� üũ
        self.assertGreater(len(soup.body.select('h2')), 1)
        
    def test_blog_list_empty(self):
        '''
        �Խù� ����� ��������� ��츦 �׽�Ʈ�մϴ�
        '''

        print('---Blog ������ ����Ʈ�� ��������� ��� �׽�Ʈ---')
        response = self.client.get('/blog/')
        soup = BeautifulSoup(response.content, 'html.parser')

        # Post ��ü�� ������ 0�̶��, �Խù� ���� �ȳ� ���� üũ
        if Post.objects.count() == 0:
            self.assertIn('�Խù��� �������� �ʽ��ϴ�. ù��° �Խù��� ���ΰ��� �Ǽ���!', soup.body.text)
        
    def test_blog_detail(self):
        '''
        �Խù� �� �������� ����, ����, ������¥ ���� �����մϴ�
        '''

        print('---Blog �� ������ �׽�Ʈ---')
        # Post �� �ֱ�
        post_001 = Post.objects.create(
            title= 'ù��° �Խù� �����Դϴ�',
            content= 'ù��° �Խù� �����Դϴ�',
        )
        response = self.client.get('/blog/1')
        soup = BeautifulSoup(response.content, 'html.parser')

        # class �̸��� ���� ����, ����, ������¥ ��������
        title = soup.select('.contents-heading')[0].text
        content = soup.select('.contents-text')[0].text
        updated_at = soup.select('.contents-updated')[0].text

        # ���� ���� ��, ������¥�� ��Ȯ�� �� �� ���� ������ �� �������� �Ǵ�
        self.assertIn('ù��° �Խù� �����Դϴ�', title)
        self.assertIn('ù��° �Խù� �����Դϴ�', content)
        self.assertTrue(updated_at)

    def test_blog_detail_template(self):
        '''
        �Խù� �� ������ �� base.html ��ӿ� ���� �׽�Ʈ�� �����մϴ�
        '''
        print('---Blog �� ������ ��� �׽�Ʈ---')
        # Post �� �ֱ�
        post_001 = Post.objects.create(
            title= 'ù��° �Խù� �����Դϴ�',
            content= 'ù��° �Խù� �����Դϴ�',
        )
        response = self.client.get('/blog/1')
        soup = BeautifulSoup(response.content, 'html.parser')

        # ���, �ٵ�, Ǫ�� �±� �������� �Ǵ�
        self.assertTrue(soup.find('header'))
        self.assertTrue(soup.find('body'))
        self.assertTrue(soup.find('footer'))
```

## Test ����
```
python manage.py test
```

## Test ���
![test](https://github.com/Nam-Younghoon/travelog/assets/58909988/ce37adb8-4656-4d1e-923a-b57ebe54f73f)