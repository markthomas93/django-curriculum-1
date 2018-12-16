# Python

## Windows 설치
https://docs.djangoproject.com/ko/2.1/howto/windows/

## Homebrew 설치
https://brew.sh/index_ko
```sh
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## Python3 설치
```sh
brew install python3
```

## pip (Python Index Package) 설치
https://pip.pypa.io/en/stable/installing/#upgrading-pip
```sh
# pip 버전 올리기 (현재 버전 18.1)
python3 -m pip install -U pip
```

## pylint 설치
```sh
python3 -m pip install -U pylint --user
```

# Django

## Django 추천 버전
https://docs.djangoproject.com/ko/2.1/faq/install/#faq-python-version-support

## Django 설치
https://www.djangoproject.com/download/
```sh
pip install Django==2.1.3
```

## Django 확인
https://docs.djangoproject.com/ko/2.1/intro/install/
```sh
python3
>>>
import django
print(django.get_version())
```

```sh
python3 -m django --version
```

## 프로젝트 생성
https://docs.djangoproject.com/ko/2.1/intro/tutorial01/
```sh
django-admin startproject django_tutorial
```

## .gitignore
```gitignore
.DS_Store

# Editor directories and files
.idea
.vscode
*.suo
*.ntvs*
*.njsproj
*.sln
*.sw*

# __pycache__
__pycache__

# db.sqlite3
db.sqlite3

```

## 서버 실행
```sh
python3 manage.py runserver
```

## Polls 앱 생성
```sh
python3 manage.py startapp polls
```

## Polls 앱 라우터 설정
/polls/views.py
```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")

```

/polls/urls.py
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index')
]

```

/django_tutorial/urls.py
```diff
- from django.urls import path
+ from django.urls import include, path
```

```python
    path('polls/', include('polls.urls')),
```

## Database Setting
https://docs.djangoproject.com/ko/2.1/ref/settings/#std:setting-DATABASES

### MySQL DB API Drivers
https://docs.djangoproject.com/ko/2.1/ref/databases/#mysql-db-api-drivers

```sh
pip install mysqlclient
```

#### 윈도우에서 설치가 안될 경우
http://lemontia.tistory.com/756

https://www.lfd.uci.edu/~gohlke/pythonlibs/#mysqlclient

python이 32비트인지 64인지 확인 후에 mysqlclient-1.3.13-cp37-cp37m-win32.whl 다운로드

```cmd
# 다운 받은 경로에서
pip install mysqlclient-1.3.13-cp37-cp37m-win32.whl
```

/django_tutorial/settings.py
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'mydatabase',
        'USER': 'mydatabaseuser',
        'PASSWORD': 'mypassword',
        'HOST': '127.0.0.1',
        'PORT': '3306',
    }
}
```


## Migrate 실행
https://docs.djangoproject.com/ko/2.1/intro/tutorial02/
```sh
python3 manage.py migrate

# mysql>
SHOW DATABASES;
USE DJANGO_TUTORIAL;
SHOW TABLES;
# mysql>
```

## Polls 모델 생성
/polls/models.py
```python
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
      # models.CASCADE는 부모인 Question 레코드가 지워지면 자식인 Choice 레코드 모두를 지우겠다는 뜻이다.
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

```

## Polls 모델 활성화
/django_tutorial/settings.py
```python
INSTALLED_APPS = [
    'polls.apps.PollsConfig',
```

migrations 등록
```sh
python3 manage.py makemigrations polls
```

실행될 SQL문 확인
```sh
python3 manage.py sqlmigrate polls 0001
```

Migrate 실행
```sh
python3 manage.py migrate
```

## Terminal에서 django 명령 실행 시키기
```sh
# /django_tutorial/settings.py 파일을 import 하고 python을 실행한 것과 같다.
python3 manage.py shell
```

```python
from polls.models import Choice, Question

# question 개수 0개
Question.objects.all()

from django.utils import timezone

q = Question(question_text="What's new?", pub_date=timezone.now())
q
q.save() # DB 해당 레코드 생성

# mysql>
SELECT * FROM polls_question;
# mysql>

q.id
q.question_text
q.pub_date
q.question_text = "What's up?"
q.save()

# mysql>
SELECT * FROM polls_question;
# mysql>

# question 개수 1개
Question.objects.all()
```

### Polls 모델에서 사용가능한 함수 추가
/polls/models.py
```python
import datetime
from django.utils import timezone

class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text

```

```sh
python3 manage.py shell
```

```python
from polls.models import Choice, Question

Question.objects.all()
Question.objects.filter(id=1)
Question.objects.filter(question_text__startswith='What')

from django.utils import timezone
current_year = timezone.now().year
Question.objects.get(pub_date__year=current_year)

Question.objects.get(id=2) # 에러 발생

Question.objects.get(pk=1)
q = Question.objects.get(pk=1)
q.was_published_recently()

q.choice_set.all()
q.choice_set.create(choice_text='Not much', votes=0)

# mysql>
SELECT * FROM polls_choice;
# mysql>

q.choice_set.create(choice_text='The sky', votes=0)
c = q.choice_set.create(choice_text='Just hacking again', votes=0)
c.question

q.choice_set.all()
q.choice_set.count()
Choice.objects.filter(question__pub_date__year=current_year)

c = q.choice_set.filter(choice_text__startswith='Just hacking')
c.delete() # DB 해당 레코드 삭제
```

## Polls Admin에 등록

### Admin user 등록
```sh
python3 manage.py createsuperuser
```
```sql
SELECT * FROM auth_user;
```

/polls/admin.py
```python
from .models import Question

admin.site.register(Question)
```

http://127.0.0.1:8000/admin

## Polls views.py 라우터 등록
/polls/views.py
```python
def detail(request, question_id):
    return HttpResponse("You're lokking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)

```

/polls/urls.py
```python
urlpatterns = [
    # ex: /polls/
    path('', views.index, name='index'),
    # ex: /polls/5/
    path('<int:question_id>/', views.detail, name='detail'),
    # ex: /polls/5/results/
    path('<int:question_id>/results/', views.results, name='results'),
    # ex: /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),
]

```

## Polls template view 적용
/polls/views.py
```python
from django.template import loader
from .models import Question

def index(request):
    lastes_question_list = Question.objects.order_by('-pub_date')[:5]
        # [:5] = 정렬순서에서 5개까지만 읽는다.
    template = loader.get_template('polls/index.html')
    context = {
        'latest_question_list': lastes_question_list,
    }
    return HttpResponse(template.render(context, request))

```

/polls/templates/polls/index.html
```python
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}

```

## Polls shortcuts render 적용
/polls/views.py
```diff
- from django.template import loader
- template = loader.get_template('polls/index.html')
- return HttpResponse(template.render(context, request))
+ return render(request, 'polls/index.html', context)
```

## Polls http404
/polls/views.py
```python
from django.http import Http404

def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404('Question does not exist')
    return render(request, 'polls/detail.html', {'question': question})
```

/polls/template/detail.html
```python
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>

```

## Polls shortcuts 404
/polls/views.py
```diff
- from django.shortcuts import render
- from django.http import Http404
+ from django.shortcuts import get_object_or_404, render

- try:
-     question = Question.objects.get(pk=question_id)
- except Question.DoesNotExist:
-     raise Http404('Question does not exist')
+ question = get_object_or_404(Question, pk=question_id)
```

## Polls remove URL: /polls
/polls/template/index.html
```diff
- <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
+ <li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

/polls/urls.py
```diff
- path('<int:question_id>/', views.detail, name='detail'),
+ path('detail/<int:question_id>/', views.detail, name='detail'),
    # 되는지 확인만 해보기
```

## Polls route namespace 적용
/polls/urls.py
```python
app_name = 'polls'
# urlpatterns = [ ...
```

/polls/template/index.html
```diff
- <li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
+ <li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```

## Polls detail form 만들기
/polls/template/detail.html
```python
<h1>{{ question.question_text }}</h1>

{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
{% endfor %}
<input type="submit" value="Vote">
</form>

```

## Polls vote, results 만들기
/polls/views.py
```diff
- from django.http import HttpResponse
- from .models import Question
+ from .models import Choice, Question
+ from django.http import HttpResponseRedirect
+ from django.urls import reverse
```
```python
def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})

def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))

```

/polls/template/results.html
```python
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>

```

## Polls generic view
/polls/views.py 안에 함수를 클래스화 한다.

/polls/urls.py
```diff
- path('', views.index, name='index'),
+ path('', views.IndexView.as_view(), name='index'),

- path('<int:question_id>/', views.detail, name='detail'),
+ path('<int:pk>/', views.DetailView.as_view(), name='detail'),

- path('<int:question_id>/results/', views.results, name='results'),
+ path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
```

/polls/views.py
```python
from django.views import generic

# def index(request):
# def detail(request, question_id):
# def results(request, question_id):

class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]

class DetailView(generic.DeleteView):
    model = Question
    template_name = 'polls/detail.html'

class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'

```

## Polls make test bug
/polls/tests.py
```python
import datetime
from django.utils import timezone
from .models import Question

class QuestionModelTests(TestCase):
    def test_was_published_recently_with_future_question(self):
        """
        was_published_recently() returns False for questions whose pub_date
        in in the future
        """
        time = timezone.now() + datetime.timedelta(days=30)
        future_question = Question(pub_date=time)
        self.assertIs(future_question.was_published_recently(), False)

```

실행
```sh
python3 manage.py test polls
```

## Polls fixup was_published_recently
/polls/models.py
```diff
- return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
+ now = timezone.now()
+ return now - datetime.timedelta(days=1) <= self.pub_date <= now
```

실행
```sh
python3 manage.py test polls
```

## Polls test more
/polls/tests.py
```python
    def test_was_published_recently_with_old_question(self):
        """
        was_published_recently() returns False for questions whose pub_date
        is older tahn 1 day.
        """
        time = timezone.now() - datetime.timedelta(days=1, seconds=1)
        old_question = Question(pub_date=time)
        self.assertIs(old_question.was_published_recently(), False)

    def test_was_published_recently_with_recent_question(self):
        """
        was_published_recently() returns True for questions whose pub_date
        is within the last day.
        """
        time = timezone.now() - datetime.timedelta(hours=23, minutes=59, seconds=59)
        recent_question = Question(pub_date=time)
        self.assertIs(recent_question.was_published_recently(), True)

```

실행
```sh
python3 manage.py test polls
```

## Polls view 개선시키기
/polls/views.py
```python
from django.utils import timezone

class IndexView(generic.ListView):
    # ...
    def get_queryset(self):
        """
        Return the last five published questions (not including those set to be
        published in the future).
        """
        return Question.objects.filter(
            pub_date__lte=timezone.now()
        ).order_by('-pub_date')[:5]

class DetailView(generic.DeleteView):
    # ...
    def get_queryset(self):
        """
        Excludes any questions that aren't published yet.
        """
        return Question.objects.filter(pub_date__lte=timezone.now())

```

/polls/tests.py
```python
from django.urls import reverse

def create_question(question_text, days):
    """
    Create a question with the given `question_text` and published the
    given number of `days` offset to now (negative for questions published
    in the past, positive for questions that have yet to be published).
    """
    time = timezone.now() + datetime.timedelta(days=days)
    return Question.objects.create(question_text=question_text, pub_date=time)

class QuestionIndexViewTests(TestCase):
    def test_no_questions(self):
        """
        If no questions exist, an appropriate message is displayed.
        """
        response = self.client.get(reverse('polls:index'))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "No polls are available.")
        self.assertQuerysetEqual(response.context['latest_question_list'], [])

    def test_past_question(self):
        """
        Questions with a pub_date in the past are displayed on the
        index page.
        """
        create_question(question_text="Past question.", days=-30)
        response = self.client.get(reverse('polls:index'))
        self.assertQuerysetEqual(
            response.context['latest_question_list'],
            ['<Question: Past question.>']
        )

    def test_future_question(self):
        """
        Questions with a pub_date in the future aren't displayed on
        the index page.
        """
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse('polls:index'))
        self.assertContains(response, "No polls are available.")
        self.assertQuerysetEqual(response.context['latest_question_list'], [])

    def test_future_question_and_past_question(self):
        """
        Even if both past and future questions exist, only past questions
        are displayed.
        """
        create_question(question_text="Past question.", days=-30)
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse('polls:index'))
        self.assertQuerysetEqual(
            response.context['latest_question_list'],
            ['<Question: Past question.>']
        )

    def test_two_past_questions(self):
        """
        The questions index page may display multiple questions.
        """
        create_question(question_text="Past question 1.", days=-30)
        create_question(question_text="Past question 2.", days=-5)
        response = self.client.get(reverse('polls:index'))
        self.assertQuerysetEqual(
            response.context['latest_question_list'],
            ['<Question: Past question 2.>', '<Question: Past question 1.>']
        )

class QuestionDetailViewTests(TestCase):
    def test_future_question(self):
        """
        The detail view of a question with a pub_date in the future
        returns a 404 not found.
        """
        future_question = create_question(question_text='Future question.', days=5)
        url = reverse('polls:detail', args=(future_question.id,))
        response = self.client.get(url)
        self.assertEqual(response.status_code, 404)

    def test_past_question(self):
        """
        The detail view of a question with a pub_date in the past
        displays the question's text.
        """
        past_question = create_question(question_text='Past Question.', days=-5)
        url = reverse('polls:detail', args=(past_question.id,))
        response = self.client.get(url)
        self.assertContains(response, past_question.question_text)

```

## Polls static assetes
/polls/templates/polls/index.html
```python
{% load static %}
<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}">
```

/polls/static/polls/style.css
```css
body {
  background: white url("images/background.gif") no-repeat;
}
/* 이미지 파일 /polls/static/polls/images/background.gif */
li a {
  color: green;
}

```

재시작 하고 확인

## Polls Admin fields order change
/polls/admin.py
```python
class QuestionAdmin(admin.ModelAdmin):
    fields = ['pub_date', 'question_text']

```
```diff
- admin.site.register(Question)
+ admin.site.register(Question, QuestionAdmin)
```

Admin Question 확인

```python
class QuestionAdmin(admin.ModelAdmin):
    # fields = ['pub_date', 'question_text']
    fieldsets = [
        (None, {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date']}),
    ]

```

Admin Question 확인

## Polls Admin register Choice
```diff
- from .models import Question
+ from .models import Choice, Question

admin.site.register(Choice)

```

## Polls Admin Question에서 Choice 등록 하기
/polls/admin.py
```python
# class ChoiceInline(admin.StackedInline):
class ChoiceInline(admin.TabularInline):
    model = Choice
    extra = 3

class QuestionAdmin(admin.ModelAdmin):
    # ...
    inlines = [ChoiceInline]
```
```diff
- admin.site.register(Choice)
```

## Polls Admin Question change list
/polls/admin.py
```python
class QuestionAdmin(admin.ModelAdmin):
    # ...
    list_display = ('question_text', 'pub_date', 'was_published_recently')
    list_filter = ['pub_date']
    search_fields = ['question_text']
```

/polls/models.py
```python
class Question(models.Model):
    # ...
    was_published_recently.admin_order_field = 'pub_date'
    was_published_recently.boolean = True
    was_published_recently.short_description = 'Published recently?'
```

## Admin change template path
/django_tutorial/settings.py
```diff
TEMPLATES = [
    {
-        'DIRS': [],
+        'DIRS': [os.path.join(BASE_DIR, 'templates')],

```

Django 소스 파일 경로 찾기
```sh
python3 -c "import django; print(django.__path__)"
```

Django Admin 소스 파일 경로 이동
```sh
cd {해당경로}/contrib/admin/templates/admin
open .
```

/templates/admin 폴더 생성 후 복사하기

/templates/admin/base_site.html
```html
<!-- <h1 id="site-name"><a href="{% url 'admin:index' %}">{{ site_header|default:_('Semin administration') }}</a></h1> -->
<h1 id="site-name"><a href="{% url 'admin:index' %}">Semin admin</a></h1>
```
