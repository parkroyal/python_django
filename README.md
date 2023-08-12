# python_django
python_django


pip install django

django-admin startproject mysite

py manage.py startapp polls

polls 
admin.py : 設定資料庫呈現的模式，之後會跟models溝通
models.py : 建構你的資料庫型態
tests.py : 這是拿來檢查商業邏輯的地方，也就是用來測試你的邏輯是否有遺漏，這裡我們沒有要討論太多這方面的議題，但是你要記住，寫測試是一件相當重要的事情，千萬不能小看使用者的潛能
views.py : 相信你還沒有忘記，在 Day1我們這一位擔任控制者(controller)的身分，沒錯! 它就是寫商業邏輯的地方，它會跟urls.py做呼應，並將所需傳達給前端
urls.py : 它擔任著橋樑的角色，讓views.py與相對的網站做對應。蛤? 你說怎麼沒見到 urls.py，因為我們要自己建阿^^，在這裡建議可以把內部 ithome的urls.py複製貼過來就好，但是內容要修改!! 待會再來說明修改的部分
apps.py : 這裡你只要先了解，這是用來區別app的一個檔案即可
init.py : 相信大家都還沒有忘記^^，就是告訴Python這資料夾是個套件
migrations : 這資料夾裡面存放的內容，記錄著models裡面所創建的資料庫型態，

1. 創建views 
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")

2. 創建URL 映射該view
在polls下創建urls.py

from django.urls import path

from . import views

urlpatterns = [
    path("", views.index, name="index"),
]


3.在URLconf文件中創建指定我们创建的 polls.urls 模块。在 mysite/urls.py 文件的 urlpatterns 列表里插入一个 include()
urlpatterns = [
    path("polls/", include("polls.urls")),
]

path(route='', view=include('polls.urls'), kwargs=None, name=None)


# part2. database
mysite/setting.py

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

django.db.backends.mysql

## INSTALLED_APPS
django.contrib.admin -- 管理员站点， 你很快就会使用它。
django.contrib.auth -- 认证授权系统。
django.contrib.contenttypes -- 内容类型框架。
django.contrib.sessions -- 会话框架。
django.contrib.messages -- 消息框架。
django.contrib.staticfiles -- 管理静态文件的框架。

## 開啟預設的應用程式(上面那些)
python manage.py migrate

## 創建model

### 定義polls裡面的models

问题 Question 和选项 Choice
mysite\polls\models.py

```
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

### 安裝模型至APP中
mysite\mysite\settings.py
INSTALLED_APPS 增加    "polls.apps.PollsConfig",

```
執行makemigrations
python manage.py makemigrations polls
Migrations for 'polls':
  polls\migrations\0001_initial.py
    - Create model Question
    - Create model Choice
```
Django 会检测你对模型文件的修改（在这种情况下，你已经取得了新的），并且把修改的部分储存为一次 迁移。

可以使用命令行查看遷移的具體內容
```
python manage.py sqlmigrate polls 0001
BEGIN;
--
-- Create model Question
--
CREATE TABLE "polls_question" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "question_text" varchar(200) NOT NULL, "pub_date" datetime NOT NULL);
--
-- Create model Choice
--
CREATE TABLE "polls_choice" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "choice_text" varchar(200) NOT NULL, "votes" integer NOT NULL, "question_id" bigint NOT NULL 
REFERENCES "polls_question" ("id") DEFERRABLE INITIALLY DEFERRED);
CREATE INDEX "polls_choice_question_id_c5b4b260" ON "polls_choice" ("question_id");
COMMIT;
```


python manage.py migrate

步驟:
1.编辑 models.py 文件，改变模型。
2.运行 python manage.py makemigrations 为模型的改变生成迁移文件。
3.运行 python manage.py migrate 来应用数据库迁移。


## 創建 Admin 帳號
```
python manage.py createsuperuser
```

向管理页面中加入投票应用¶
polls/admin.py

```python
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```


## part3. view

## 1. 編寫view  (polls/views.py)
```python
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)


def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)


def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

## 2. 編寫urls
```python
from django.urls import path

from . import views

urlpatterns = [
    # ex: /polls/
    path("", views.index, name="index"),
    # ex: /polls/5/
    path("<int:question_id>/", views.detail, name="detail"),
    # ex: /polls/5/results/
    path("<int:question_id>/results/", views.results, name="results"),
    # ex: /polls/5/vote/
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```

## 3. 修改views.py
### 3.1 
```python
def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    output = ", ".join([q.question_text for q in latest_question_list])
    return HttpResponse(output)
```

### 3.2 增加template資料夾
mysite\polls\templates\polls\index.html
```
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

### 3.2.1 render
```
def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    context = {"latest_question_list": latest_question_list}
    return render(request, "polls/index.html", context)
```


### 3.3 raise Http404
polls/views.py
```python
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, "polls/detail.html", {"question": question})
```
### 3.3.1 get_object_or_404
```python
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/detail.html", {"question": question})
```


## 3.4 使用模板系統

### 3.4.1 去除模板中的硬编码 URL (看不懂)

polls/templates/polls/index.html
```html
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```

{% url %} 模板标签，它能够为我们做这件事情。修改 polls/templates/polls/index.html 模板，使其使用 {% url %} 模板标签：
```html
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

### 3.4.2 命名空间
polls/urls.py
```python
app_name = "polls"
```

index.html
```html
url 'detail' -> url 'polls:detail'
```


# part4. 表單處理