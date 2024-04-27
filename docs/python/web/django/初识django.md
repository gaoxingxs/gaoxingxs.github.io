## 使用django创建项目

**安装django**

```bash
python -m pip install Django
```

**验证**

```
>>>
>>> import django
>>> print(django.get_version())
5.0.4
```

**创建项目**

安装完django后，会自动将`django-admin`命令添加到系统环境变量中。

```
django-admin startproject mysite
```

[django-admin 和 manage.py | Django 文档 | Django](https://docs.djangoproject.com/zh-hans/5.0/ref/django-admin/)

## 应用

前面创建的是一个项目，一个项目是由一个或多个应用组成。项目中的每个应用都是独立的，可以拥有独立的数据库、模板文件、业务代码。比如，一个网站，可以分为前台用户应用和后台管理员应用。

**创建应用**

```
python manage.py startapp app01
```

**MTV模型**

![alt text](<../../../assets/images/Pasted image 20240406210147.png>)

创建完应用后需要将应用配置到项目的setting中

setting.py
```python
INSTALLED_APPS = [
    ……
    "helloword.apps.HellowordConfig",
]
```

## URL

### URL模块化

在项目中，每个app下面会有一个`urls.py`，在这个文件总映射此app自己的视图。在项目的`urls.py`中借助`include()`函数，统一包含每个`app`的`urls`。

项目urls.py
```python
from django.contrib import admin
from django.urls import include, path


urlpatterns = [
    path("book/", include("book.urls")),
    path("admin/", admin.site.urls),
]
```

在请求book相关的url的时候，都需要加一个book前缀。

### URL传参

在url中携带参数有两种方式：

1、query string
```python
# views.py

# 请求url：http://localhost:8000/book/detail_query_string/?book_id=2
def detail_query_string(request):
    # request.GET: {"id: "3"}
    book_id = request.GET.get('book_id')
    return HttpResponse(f"Hello, world. You're at the polls detail view. book_id={book_id}")

# urls.py
urlpatterns = [
    ...
    path("detail_query_string/", views.detail_query_string, name="detail_query_string"),
]
```

2、path
```python
# views.py

# path参数需要在views中声明，且名称要和urls中定义的一致。
def detail_path(request, book_id):
    return HttpResponse(f"Hello, world. You're at the polls detail view. book_id={book_id}")

# urls.py
path("detail_path/<book_id>", views.detail_path, name="detail_path"),
# 可以定义类型，如果参数不能转为定义的类型，会出现404错误
# 除int类型外，还支持str、slug、uuid、path类型
# str：默认的转换器，非空的字符串类型，不能包含斜杠/
# int：匹配任意的零或者整数的整形
# slug：由英文中的-或者_连接英文字符或者数字而形成的字符串
# uuid：匹配uuid字符串
# path：匹配非空的英文字符串，可以包含斜杠/
path("detail_path2/<int:book_id>", views.detail_path2, name="detail_path2"),
```

### path函数

path函数的定义为：`path(route, view, name=None, kwargs=None)`

- route：url的匹配规则，可以通过`<>`指定path参数
- view：可以为一个`视图函数`或`类试图.as_view()`或`include()函数的返回值`
- name参数：给这个url取名，在项目比较大时，比较有用。
- kwargs参数：传递额外参数

### 命名空间

命名空间有助于避免不同应用间的 URL 冲突。

1、在应用的urls.py中使用`app_name`参数为url指定一个命名空间。

```python
app_name = 'book'  # 定义名称空间

urlpatterns = [
    path('some_view/', views.some_view, name='some_view'),
    # 其他 URL 模式...
]
```

2、在项目的urls.py中，使用 include 函数将应用的 urls.py 文件包含到项目的主 urls.py 文件中，并使用 app_name 参数指定名称空间。

```python
from django.contrib import admin
from django.urls import include, path


urlpatterns = [
    path("book/", include("book.urls", namespace="book")),
    path("admin/", admin.site.urls),
]
```

3、当你在模板中使用 {% url %} 模板标签时，可以通过名称空间来引用特定的 URL。

```python
<a href="{% url 'some_view' with namespace='book' %}">View Blog Posts</a>
```

4、在视图中使用名称空间的 URL：

```python
url = f"{reverse("book:detail_query_string")}?book_id=1"
```
### url反转

通过path函数的name反转得到url。

```python
# 反转带path参数的url
url = reverse("detail_path", kwargs={"book_id": "1"})

# 反转查询字符串参数的url，只能通过字符串拼接的方式
url = f"{reverse("detail_query_string")}?book_id=1"

# 如果由应用命名空间或实例命名空间，那么应该在反转前加上命名空间
url = reverse("book:detail_path", kwargs={"book_id": "1"})
```

## 模板

主流模板引擎有`DjangoTemplates`和`jinja2`。

### 模板配置

setting.py
```python
# 模板配置
TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",  # 定义模板引擎
        # 设置模板所在的路径
        # 如果多个路径中的模板文件名称相同，会优先使用前面的
        "DIRS": [os.path.join(BASE_DIR, "templates")],
        "APP_DIRS": True,  # 是否在APP里查找模板文件
        "OPTIONS": {  # 用于填充在RequestContext的上下文（模板里面的变量和指令），一般情况下不做任何修改
            "context_processors": [
                "django.template.context_processors.debug",
                "django.template.context_processors.request",
                "django.contrib.auth.context_processors.auth",
                "django.contrib.messages.context_processors.messages",
            ],
        },
    },
]
```


### 模板渲染
views.py
```python
from django.shortcuts import render


def template(request):
    return render(request, "index.html")
```

### 模板语法

#### 渲染变量

```python
def template(request):
    username = "tom"
    book = {"name": "水浒传"}
    books = [
        {"name": "红楼梦"},
        {"name": "西游记"},
        {"name": "三国演义"},
    ]
    context = {
        "username": username,
        "book": book,
        "books": books,
        "age": 18,
    }
    return render(request, "book/index.html", context=context)
```

```html
{#模板中不支持使用book["name"]语法获取字典中的数据#}
<h1>图书名称：{{ book.name }}</h1>
{#遍历列表#}

一、遍历列表
<ul>
{% for item in books %}
	{# forloop.counter从1开始 forloop.counter0从0开始 #}
	<li>{{ forloop.counter0 }}{{ item.name }}</li>
{% endfor %}
</ul>


二、遍历字典
{% for key, value in book.items %}
	<p>key:{{ key }},value: {{ value }}</p>
{% endfor %}


三、for in empty标签 遍历对象为空时，会执行empty中的内容
{% for key, value in None %}
	<p>key:{{ key }},value: {{ value }}</p>
{% empty %}
	<p>什么都没有。。。</p>
{% endfor %}


四、if标签
{% if age > 18 %}
	<p>可以进入网吧</p>
{% elif age == 18 %}
    <p>刚满18</p>
{% else %}
    <p>未满18</p>
{% endif %}


五、with标签：有时候一个变量访问的时候比较复习，那么可以给这个变量去一个别名
注意：1、with语句定义的变量，只能在{% with %}{% endwith %}中使用，不能在这个表亲啊外面使用。
	2、定义变量的时候，不能在等号两边留有空格
{% with name=books.0.name %}
    <p>{{ name }}</p>
{% endwith %}

{% with books.0.name as name %}
    <p>{{ name }}</p>
{% endwith %}


六、url标签：在模板中经常需要url跳转，直接把url写死对于维护不太方便，可以通过url标签用配置的name参数进行反转来实现。
{#path("detail_path/<book_id>/<book_name>", views.detail_path, name="detail_path"),#}
{#指定path参数时等号两边不能有空格#}
<a href="{% url 'book:detail_path' book_id=1 book_name='三国' %}">click me</a>

{#对于路径参数，要手动拼接#}
<a href="{% url 'book:detail_query_string' %}?book_id=1">click me</a>



{#通过下表获取列表中的元素#}
{{ books.0.name }}
{{ books.1.name }}
```

#### 常用过滤器

1、add
会先将值和参数转为整形相加，如果转换整形的过程中失败了，那么会进行拼接，如果是字符串，那么会拼接为字符串，如果是列表，那么会拼接形成一个新列表。

```
{{ value|add:"2" }}
```

如果value是数值类型，则进行加法操作；如果value是一个字符串，则进行字符串拼接。

2、cut
相当与python中的`replace(args, "")`

```
{{ value|cut:"a" }}
```

3、date
将日期按照指定的格式，格式化为字符串。

```
{{ birthday|date:"Y-m-d H:i:s" }} //2024-04-10 20:37:01
```

4、default
如果被评估为False，比如：`[] "" None {} False`等这些在if判断中为False的值，都会使用default过滤器提供的默认值。

```
{{ test|default:"什么都没有" }}
```

5、default_if_none
如果值为None，会使用提供的默认值。

6、first
返回列表、元组、字符串中的第一个元素。

7、first
返回列表、元组、字符串中的最后一个元素。

8、floatformat
使用四舍五入的方式格式化一个浮点类型。没有传递参数时，只会在小数点后保留一位小数。如果小数后面全是0，那么只会保留整数。也可以传递一个参数，标识要保留几位小数。

![floatformaat格式](<../../../assets/images/Pasted image 20240410205347.png>)

9、join
与python中的join类似。

10、length
获取列表、元组、字符串、字典的长度，如果值为None，那么以上将返回0。

11、lower
将字符全部转为小写。

12、upper
将字符全部转为大写。

13、random
在列表、字符串、元组中随机选择一个值。

14、safe
对html标签进行解析。

```
context = {
		"html": "<h1>hello</h1>"
	}

{{ html }} // 页面上会原封不动的显示<h1>hello</h1>

{{ html|safe }} //会解析h1，页面上显示h1样式的hello
```

15、slice
类似于Python中的切片操作。

```
{{ value|slice:"2:" }}
```

16、stringtags
删除字符串中所有的html标签。比如value是`<h1>hello word</h1>`，下面的代码将会输出hello word。

```
{{ value|slice:"2:" }}
```

17、truncatechars
如果给定的字符串长度超过了过滤器指定的长度，那么就好不i进行切割，并且会拼接三个点来作为省略号。

18、truncatechars_html
与truncatechars类似，不会对html标签进行切割，只会切割html标签中的内容。如果value是`<p>hello word</p>`，将输出`<p>hel...</p>`

### 模板结构

#### include

需要被引入的html
```html
<div>
    <h2>热门文章</h2>
    <ul>
        <li>文章1</li>
        <li>文章2</li>
    </ul>
</div>
```

主html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
{% include "book/xfz_hotarticle.html" %}
</body>
</html>
```

include引入的模板，会自动使用主模板的上线问，也即可以自动的使用主模板的变量。如果想要传入一些其他的参数，那么可以使用with语句：
```
{% include "book/xfz_hotarticle.html" with username="tom" %}
```

#### 模板继承extend

在前端网页开发中，有些代码是需要复用的，可以使用include标签来实现，也可以使用比较强大模板继承来实现。模板继承可以在父模板中先定义好一些子模版需要用到的代码，然后在子模版中直接继承就可以了。并且因为子模版肯定有自己的不同代码，因此可以在父模板中定义一个block接口，然后子模版再去实现。

父模板代码：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>
        {% block title %}{% endblock %}
    </title>
    {% block style %}{% endblock %}
</head>
<body>
<header>
    <ul>
        <li>首页</li>
        <li>创业课堂</li>
    </ul>
</header>
{#需要子模版自己去实现的代码#}
{% block body %}
{% endblock %}
<footer>@2022 XXX有限公司 京ICO备001号</footer>
</body>
</html>

```

子模版代码：
```html
{% extends "book/xfz_base.html" %}
{#子模版自己的代码#}
{% block title %}
    首页
{% endblock %}

{% block style %}
    <style>
        body {
            background: pink;
        }
    </style>
{% endblock %}

{% block body %}
    {% include "book/xfz_hotarticle.html" %}
{% endblock %}
```

注意：
1、extends标签必须放在子模板的第一行，子模板的代码必须放在block中，且块名称和父模板一致。
2、子模板的block会覆盖父模板同名的block块。如果想要子模块不覆盖父模块的内容，而是新增，可以使用 block.super。
父模板：
```
<head>
    <meta charset="UTF-8">
    <title>
        {% block title %}首页{% endblock %}
    </title>
</head>
```

子模版：
```
{% block title %}
    {{ block.super }}
{% endblock %}
```

### 加载静态资源

1、首先完成静态资源的配置
2、在模板中使用`load`标签加载`static`标签。
```html
{% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>
        {% block title %}
            首页
        {% endblock %}
    </title>
    <link rel="stylesheet" href="{% static "css/index.css" %}">
    <script src="{% static "js/index.js" %}"></script>
    {% block style %}{% endblock %}
</head>
<body>
</body>
</html>
```

3、如果不想每次在模板中加载静态文件都是用load标签加载static标签，可以在setting.py中的TEMPLATES/OPTIONS添加`"builtins": ["django.templatetags.static"]`

```python
TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",  # 定义模板引擎
        # 设置模板所在的路径
        # 如果多个路径中的模板文件名称相同，会优先使用前面的
        "DIRS": [os.path.join(BASE_DIR, "book/../templates")],
        "APP_DIRS": True,  # 是否在APP里查找模板文件
        "OPTIONS": {  # 用于填充在RequestContext的上下文（模板里面的变量和指令），一般情况下不做任何修改
            ...
            "builtins": ["django.templatetags.static"],
        },
    },
]
```

## 静态资源配置

### 静态资源配置

`STATIC_URL`:用于定义静态文件在生产环境中的 URL 路径。当 Django 开发服务器运行时，它会通过这个 URL 路径提供静态文件。在生产环境中，通常由 Web 服务器（如 Nginx 或 Apache）来提供这些文件。

`STATICFILES_DIRS`:用于定义 Django 收集静态文件时应该搜索的目录。这些目录通常是应用的 static 目录，但也可以是项目中的其他目录或外部目录。

`STATIC_ROOT`:运行`python manage.py collectstatic` 命令时，Django 会将所有静态文件收集到这个目录下。这个目录通常不在版本控制中，因为它只是用来存放收集后的文件。在实际部署时，你可能需要将 STATIC_ROOT 目录下的文件复制到 Web 服务器的静态文件目录中，或者配置 Web 服务器（如 Nginx 或 Apache）来直接从 STATIC_ROOT 目录提供文件。

静态文件的处理方式在开发环境和生产环境中是不同的。在开发环境中，Django 会实时提供静态文件；而在生产环境中，通常会使用专门的 Web 服务器来提供这些文件，以提高性能和效率。

### 媒体文件（用户上传的文件）配置

`MEDIA_ROOT`：表示用户上传的媒体文件存储的真实目录路径（文件系统路径），例如 /path/to/your/project/media/。Django 将用户上传的文件存储在这个目录下，并且通过 `MEDIA_URL`配置的URL地址可以在网页上访问这些文件。

`MEDIA_URL`：表示用户访问媒体文件的 URL 地址，当用户上传了一张图片，该图片将会存储在 MEDIA_ROOT 目录下，并且可以通过 MEDIA_URL 配置的URL地址进行访问。

这两个配置的作用是处理用户上传的媒体文件，与静态文件（如 CSS、JavaScript 文件）的处理方式是不同的，静态文件通常由 Django 的`STATIC_URL`和`STATIC_ROOT`配置来处理。

```python title="urls.py"
urlpatterns = [
                  path("book/", include("book.urls", namespace="book")),
                  path("admin/", admin.site.urls),
              ] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

```python title="setting.py"
MEDIA_DIR = BASE_DIR / "media"
MEDIA_ROOT = "/media"
```

静态文件和媒体文件，最好是通过Nginx等专业的web服务器来部署，以上方式仅在开发阶段使用。


### 模板配置

```python
# 模板配置
TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",  # 定义模板引擎
        # 设置模板所在的路径
        # 如果多个路径中的模板文件名称相同，会优先使用前面的
        "DIRS": [os.path.join(BASE_DIR, "templates")],
        "APP_DIRS": True,  # 是否在APP里查找模板文件
        "OPTIONS": {  # 用于填充在RequestContext的上下文（模板里面的变量和指令），一般情况下不做任何修改
	        # 模板上下文处理器 作用：在模板中直接使用以下处理器返回的变量
            "context_processors": [
                "django.template.context_processors.debug",
                "django.template.context_processors.request",
                "django.contrib.auth.context_processors.auth",
                "django.contrib.messages.context_processors.messages",
            ],
        },
    },
]
```

## 数据库

### 数据库配置

[配置 | Django 文档 | Django](https://docs.djangoproject.com/zh-hans/5.0/ref/settings/#databases)

settings.py
```python
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": "mydatabase",  # 数据库名称
        "USER": "mydatabaseuser",
        "PASSWORD": "mypassword",
        "HOST": "127.0.0.1",
        "PORT": "5432",
    }
}
```

### python操作数据库

```python
from django.http import HttpResponse
from django.db import connection


def index(request):
    cursor = connection.cursor()
    cursor.execute('select * from book')
    books = cursor.fetchall()
    for book in books:
        print(book)
    return HttpResponse("Hello, world. You're at the polls index.")
```

### ORM模型

#### 定义模型

```python
from django.db import models


class Book(models.Model):
    name = models.CharField(max_length=100)
    author = models.CharField(max_length=100)
    pub_time = models.CharField(auto_add_now=True)
    price = models.CharField(default=0)
```

#### 迁移脚本文件

```
# 会在app/migrations文件夹下生成一个脚本文件
python manage.py makemigrations
```

```
# 会将模型映射到数据库中（包括django自带app的模型）
python namage.py migrate
```

#### 基本CRUD

```python
# 基本保存
def save(request):
    book = Book(name="红楼梦", author="曹雪芹", price=100)
    book.save()
    return HttpResponse("图书保存成功")

# 查询全部
def query(request):
    books = Book.objects.all()
    for book in books:
        print(book)
    return HttpResponse("查找成功")

# 过滤查找
	books = Book.objects.filter(name="红楼梦")

# 获取单条数据，当数据不存在时会抛出异常
	try:
		book = Book.objects.get(name="红楼梦1")
		print(book.name)
	except Book.DoesNotExist:
		print("图书不存在")

# 数据排序
    # 升序排序
    books = Book.objects.order_by('pub_time')
    # 降序排序
    books = Book.objects.order_by('-pub_time')


# 修改数据
Book.objects.filter(name="红楼梦").update(price=200)

# 删除数据
Book.objects.filter(name="红楼梦").delete()
```

#### 常用Field

- AutoField
	- 映射到数据库中是`int`类型，可以有自动增长的特性，如果不指定主键，那么模型会自动生成一个叫`id`的自动增长的主键。如果想指定一个其他名字的并且具有自动增长的主键，使用`AutoField`也是可以的。一般不需要使用这个类型。
- BigAutoField
	- 64位的整形，类似`AutoField`
- BooleadField
	- 模型层面接收的是True/False，数据库保存的是`tinyint`类型，没有指定默认值，默认值是None。
- CharField
	- 字符串，映射到数据库中是varchar，使用时必须指定`max_length`，最大不超过250个字符。
- DateField
	- 日期类型，可以记录年月日。映射到数据库中也是`date`类型。
	- 参数：
		- auto_now：每次修改数据的时候，都使用当前的时间，可作为update_time
		- auto_bow_add：在数据第一次被添加进去的时候，使用当前时间，可作为create_time
- DateTimeField
	- 日期时间类型，可以存储时间。映射到数据库中时`datetime`类型。也可以使用`auto_now`、`auto_now_add`两个属性。
- TimeField：
	- 时间类型，映射到数据库中是time类型。也可以使用`auto_now`、`auto_now_add`两个属性。
- EmailField
	- 会先验证是否是email格式，不是的话会报错。本质是CharField，最大长度是254个字符。
- FileField
	- 用来存储文件
- ImageField
	- 用来存储图片
- FloatField
	- 浮点类型，映射到数据库中是float类型。
- IntegerField
	- 整形，值的区间是`-2147483648-2147483647`
- BigIntegerField
	- 长整型，取值范围是`-9223372036854775808 到 9223372036854775807`
- PositiveIntegerField
	- 正整形，`0-2147483647`
- SmallIntegerField
	- 小整形，值的区间是`-32768-32767`
- TextField
	- 映射到数据库中是longtext类型
- UUIDField
	- 只能存储32位的uuid格式的字符串
- URLField
	- 类似于CharField，只能存储url格式的字符串，并且默认`max_length`长度是200

#### Field的常用参数

- null
	- 默认值是False。如果设置为True，在映射表的时候会将字段设置为可以为空。在使用字符串相关的Field（CharField/TextField）的时候就，官方推荐尽量不要使用这个参数，也就是保持默认值False。因为Django在处理字符串相关的Field的时候，即使`null=False`，django也会使用一个空的`""`来作为默认值存储进去，因此没必要设置为True
	- 如果想要在表单验证的时候允许这个字符串为空，可以勇士`blank=True`，如果使用Field是`BooleanField`，那么对应的可为空的字段为`NullBooleadField`。
- blank
	- 标识这个字段在表单验证的时候是否可为空，默认是False，上面的`null`是数据库层面的，`blank`是表单验证层面的。
- db_column
	- 字段在数据库中的名字，如果没有设置这个参数，那么将会使用模型中的名称
- default
	- 默认值。可以为一个值，也可以为一个函数，但是不支持lambda表达式，也不支持列表/字段/集合等可变的数据结构
- primary_key
	- 将字段设置为主键
- unique
	- 字段设置为唯一值

#### Meta配置

[模型 Meta 选项 | Django 文档 | Django](https://docs.djangoproject.com/zh-hans/5.0/ref/models/options/)

```python
class Book(models.Model):
    ...

    class Meta:
        # 配置数据库表名
        db_table = 'tb_book'
        # 配置排序方式 按名称正序排序 按发布时间倒序排序
        ordering = ['name', '-pub_time']
```

#### 外键

```python
# 外键的定义
class User(models.Model):
    user_id = models.BigAutoField(primary_key=True)
    user_name = models.CharField(max_length=100)


class Article(models.Model):
    article_id = models.BigAutoField(primary_key=True)
    article_name = models.CharField(max_length=100)

    # 会自动生成一个author_id的字段 与Author中的主键形成外键
    # author_id foreign key (author.author_id)
    # on_delete表示当相关联的数据被删除时，本模型的数据如何处理
    author = models.ForeignKey("User", on_delete=models.CASCADE)

# 插入数据
def article_test(request):
    user = User(user_name="test")
    user.save()
    article = Article(article_name='活着', author=user)
    article.save()
    return HttpResponse("测试成功")

# 获取数据
def article_test(request):
    article = Article.objects.get(pk=1)
    print(article.author.user_id)
    return HttpResponse("测试成功")
```

**注意事项**

1、如果想要引用另外一个`app`的模型作为外键，那么在传递`to`参数的，需要使用`app.module_name`进行指定。

```python
uthor = models.ForeignKey("user.User", on_delete=models.CASCADE)
```

2、如果模型中的外键引用的是模型本身，那么`to`参数可以为`self`，或者是这个模型的名字。比如在评论表中分为一级评论和二级评论，在模型中外键就可以引用自身。

**外键删除操作**

如果一个模型使用了外键，那么在对方那个模型删掉后，该模型执行什么样的操作，可以通过`on_delete`来指定。

1、`CASCADE`：级联操作，引用的外键数据删除了，该条数数据也删除。
2、`PROTECT`：受保护。只要这条数据引用了外键的那条数据，那么外键的那条数据就不能删除，除非是先删除这条数据，再删除外键数据。
3、`SET_NULL`：设置为空。如果外键的那条数据删除了，那么本条数据上就将这个字段设置为空。如果设置这个选项，前提是要设置这个字段可以为空。
4、`SET_DEFAULT`：设置默认值。。如果外键的那条数据删除了，那么本条数据上就将这个字段设置为默认值。如果设置这个选项，前提是要设置这个字段有默认值。
5、`SET()`：如果外键的那条数据被删除了，将会获取`SET()`函数中的值来作为这个外键的值。`SET()`函数可以接收一个可以调用的函数或者方法。
6、`DO_NOTHING`：不可以任何行为，一切全看数据级别的约束。

???+ tip "提示"
    以上设置是在django级别，如果数据库中的外键约束被设置为 RESTRICT（默认），那么即使 Django 模型配置了 CASCADE，数据库也不会允许删除操作，因为这会违反数据库层面的约束，需要执行迁移脚本来修改数据库的设置，以确保它与 Django 的 on_delete 设置相匹配。


#### 表关系

表关系可以分为一对一、一对多（多对一）、多对多。

**1、一对多（多对一）**

一对多的关系是通过`ForeignKey`实现的，如用户和文章的定义就是一对多的对应关系。

```python
class User(models.Model):
    user_id = models.BigAutoField(primary_key=True)
    user_name = models.CharField(max_length=100)


class Article(models.Model):
    article_id = models.BigAutoField(primary_key=True)
    article_name = models.CharField(max_length=100)
    author = models.ForeignKey("User", on_delete=models.CASCADE)

# 查询文章时即可获取用户的信息
def article_test(request):
    article = Article.objects.get(pk=1)
    print(article.author.user_id)

# 查询用户信息默认可通过`article_set`获取文章列表
# ForeignKey有一个related_name属性，设置此属性后，就可以使用该属性的值替代article_set
def one_to_many(request):
    user = User.objects.get(pk=1)
    articles = user.article_set.all()
    return HttpResponse(f"共查到{len(articles)}条数据")
```

**2、一对一**

用户可以存储在用户表和用户信息表，这两张表就是一对一的关系。实现方式：`OneToOneField`。

```python
class User(models.Model):
    user_id = models.BigAutoField(primary_key=True)
    user_name = models.CharField(max_length=100)

class UserExtension(models.Model):
    extension_id = models.BigAutoField(primary_key=True)
    address = models.CharField(max_length=100)
    birthday = models.DateField()
    user = models.OneToOneField("User", on_delete=models.CASCADE, related_name="extension")

def one_to_one(request):
	# 保存数据
    user = User.objects.create(user_name='test')
    user.save()
    user_extension = UserExtension.objects.create(address="北京市", birthday=date(1997, 1, 1), user=user)
    user_extension.save()

	# 查询数据
    user = User.objects.get(pk=4)
    extension = user.extension
    print(extension.extension_id)
    return HttpResponse("测试成功")
```


**3、多对多**

一篇文章可以有多个标签，一个标签可以被多个标签引用。所以标签和文章的关系就是多对多的关系。实现方式：`ManyToManyField`.

```python
class Article(models.Model):
    article_id = models.BigAutoField(primary_key=True)
    article_name = models.CharField(max_length=100)

    author = models.ForeignKey("User", on_delete=models.CASCADE, related_name="articles")
    tags = models.ManyToManyField("Tag", related_name="articles")


class Tag(models.Model):
    tag_id = models.BigAutoField(primary_key=True)
    tag_name = models.CharField(max_length=100)
```


使用ManyToManyField后，会在数据库中生成一张`article_tags`中间表，表中有三个字段`id`、`article_id`、`tag_id`。

#### 查询操作

在ORM中如果想看到sql语句，可以使用`查询结果.query`（只有查询结果是QuerySet时才有query属性，比如.all()方法）。

**1、exact**

精确查询。
```python
# select * from user where user_id = 1
user = User.objects.filter(user_id__exact=1)
```

**2、iexact**

使用`like`查询,并且不区分大小写，虽然是like，但是不会自己添加`%`，所以不会模糊查询。
```python
# select * from user where user_name like 'admin'
user = User.objects.filter(user_name__iexact=1)
```

**3、contains**

大小写敏感的左右模糊查询。
```python
# select * from user where user_name like '%Test2%'
user = User.objects.filter(user_name__contains="Test2")
```

**4、icontains**

大小写不敏感的左右模糊查询。
```python
# select * from user where user_name like '%Test2%'
user = User.objects.filter(user_name__icontains="Test2")
```

**5、in**

```python
```python
# select * from user where user_name in ('test1', 'test2')
user = User.objects.filter(user_name__in=['test1', 'test2'])
```

**6、gt、gte、lt、lte**

分别表示大于、大于等于、小于、小于等于。

**7、startwith、istartwith**

判断某个字段的值是否以某个值开始的，即sql中的右模糊查询。`startwith`大小写敏感，`istartwith`表示大小写不敏感。

**8、endwith、iwndwith**

判断某个字段的值是否以某个值结束的，即sql中的左模糊查询。`endwith`大小写敏感，`iendwith`表示大小写不敏感。

**9、range**

值是否在某个区间（包括左边，不包括右边），即sql中的`field between ... end ...`

**10、date、time、year、month、day、week_day**

 根据`date`、`datetime`类型的字段，可以使用这些查询方式。
 
**11、isnull**

根据是否为空进行查询。
```python
# select * from user where user_name is null
user = User.objects.filter(user_name__isnull=True)
user = User.objects.filter(user_name__isnull=False)
```

**12、regex、iregex**

正则表达式查询。iregex不区分大小写。
```python
users = User.objects.filter(user_name__regex=r"^test")
```

**13、根据关联的表进行查询**

```python
# 根据关联表查询
# 查询文章标题中包含“活”的文章所关联的用户
# SELECT "article_user"."user_id", "article_user"."user_name" FROM "article_user" INNER JOIN "article_article" ON ("article_user"."user_id" = "article_article"."author_id") WHERE "article_article"."article_name" LIKE %活% 
users = User.objects.filter(articles__article_name__contains="活")
```

#### 集合函数

在django ORM中，聚合函数是使用`aggregate`实现的。


**1、平均值**

```python
# 所有图书的平均值
from django.db.models import Avg
result = Book.objects.aggregate(Avg('privce'))
print(result)  # {"price_avg": 23.0}

# price_avg是根据field_avg规则生成的，支持自定义。
result = Book.objects.aggregate(my_avg=Avg('privce'))
print(result)  # {"my_avg": 23.0}
```

**2、count**

```python
from django.db.models import Count
result = Book.objects.aggregate(Count('privce'))
```

**3、Max、Min**

同上。

**4、Sum**

求和不能够再使用`aggregate`，要使用`annotate`
```python
from django.db.models import Sum

# bookorder为关联表设置的related_name
# values为返回的字段
resylt = Book.objects.annotate(total=Sum(bookorder__price)).values("name", "total")
```
???+ note "aggregate和annotate的区别"
    1、aggregate：返回聚合后的字段和值；
    2、annotate：在原来模型字段基础之上添加一个使用了聚合函数的字段，并且在使用聚合函数的时候，会使用当前这个模型的主键进行分组（group by）。比如上面的例子，如果使用annotate，那么将在每条图书的数据上都添加一个字段叫做`total`，计算这本书的销售总额。而如果使用aggregate，将会计算所有图书的销售总额。

#### F表达式和Q表达式

**1、F表达式**

允许你在查询集中引用模型字段，并执行字段值的操作，如增加、减少、测试是否等于另一个字段的值等。F 表达式对于更新操作特别有用，因为它们可以避免直接在 Python 中处理数据库值，从而提高效率。

比如要将公司所有员工的薪水提高1000元，如果按照正常的流程，应该是先从数据库中提取所有的员工工资到python内存中，然后修改数据，再将数据保存到数据库中。
而F表达式可以简化这个流程，不需要先把数据从数据库中提取出来，可以直接执行SQL语句，使用示例如下：

```python
from django.db.models import F
# 执行完不需要再save
Employee.objects.update(salary=F("salary")+1000)

# 给所有活跃用户的积分增加 10
User.objects.filter(is_active=True).update(points=F('points') + 10)

# 将用户上次登录时间设置为当前时间
User.objects.filter(id=user_id).update(last_login=F('last_login').date())
```

**2、Q表达式**

Q 表达式允许你构建复杂的查询，可以表示多个条件的 AND 或 OR 组合。这在你需要创建复杂的查询，如涉及多个条件的查询时非常有用。

```python
from django.db.models import Q

# 查找名字为 John 或者年龄大于 30 的用户
users = User.objects.filter(Q(first_name='John') | Q(age__gt=30))

# 查找名字为 John 且年龄大于 30 的用户
users = User.objects.filter(Q(first_name='John') & Q(age__gt=30))

# 可以链式使用 Q 对象，来创建更复杂的查询
users = User.objects.filter(
    Q(first_name='John') | Q(age__gt=30),
    Q(is_active=True),
)
```

## 常用装饰器

### @require_http_methods

表示视图允许的请求方法。

```python
from django.views.decorators.http import require_http_methods

@require_http_methods(['GET', 'POST'])
```

### @login_required

表示视图必须登录后才能访问。

```python
from django.contrib.auth.decorators import login_required
from django.urls import reverse, reverse_lazy

# 使用方式1：反转url名称
# 这里使用reverse函数，会报错，因为加载reverse函数时，还没有加载blog应用，所以使用reverse_lazy函数
@login_required(login_url=reverse_lazy('auth:login'))

# 使用方式二：直接使用login_url
@login_required(login_url='/auth/login')

# 使用方式三：在settings.py中配置LOGIN_URL = '/auth/login'，装饰器中必须要再传login_url参数
@login_required()
```

## 表单

### 简单使用

定义用于表单验证的模型：forms.py
```python
from django import forms


class MessageBoardForm(forms.Form):
    title = forms.CharField(min_length=3, max_length=50, label='标题',
                            error_messages={
                                "min_length": "最小长度不能小于3",
                                "max_length": "最大长度不能大于50",
                            })

    # widget是指定控件
    content = forms.CharField(widget=forms.Textarea, label="内容")
    email = forms.EmailField(label="邮箱")
```

views.py
```python
@require_http_methods(["GET", "POST"])
def index(request):
    if request.method == "GET":
        form = MessageBoardForm()
        context = {
            "form": form,
        }
        return render(request, 'front/index.html', context=context)
    else:
        form = MessageBoardForm(request.POST)
        # 验证
        if form.is_valid():
            # cleaned_data表示验证通过后的数据
            title = form.cleaned_data.get("title")
            content = form.cleaned_data.get("content")
            email = form.cleaned_data.get("email")
            print(title, content, email)
            return HttpResponse("提交成功")
        else:
            context = {
                "form": form,
            }
            return render(request, 'front/index.html', context=context)
```

front/index.html
```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>留言板</title>
</head>
<body>
<h1>留言板</h1>
<form action="" method="post">
    {{ form }}
    <input type="submit" value="提交">
</form>
</body>
</html>
```

???+ note "说明"
    上面例子中，在html中使用{{ form }}直接渲染表单的做法是不推荐的，直接使用html标签即可。

### 表单验证

#### 字段类型

Form模型提供了多种字段类型，每种类型都有一些可用的参数。

CharField:
	max_length: 字符串的最大长度。
	min_length: 字符串的最小长度。
	required: 指定字段是否为必填项。
	initial: 字段的初始值。
IntegerField:
	max_value: 整数的最大值。
	min_value: 整数的最小值。
	required: 指定字段是否为必填项。
	initial: 字段的初始值。
EmailField:
	max_length: 邮箱地址的最大长度。
	required: 指定字段是否为必填项。
	initial: 字段的初始值。
BooleanField:
	required: 指定字段是否为必填项。
	initial: 字段的初始值。
DateField:
	required: 指定字段是否为必填项。
	initial: 字段的初始值。
	input_formats: 日期输入的格式。
DateTimeField:
	required: 指定字段是否为必填项。
	initial: 字段的初始值。
	input_formats: 日期时间输入的格式。
DecimalField:
	max_value: 数值的最大值。
	min_value: 数值的最小值。
	max_digits: 数值的最大位数。
	decimal_places: 小数点后的位数。
	required: 指定字段是否为必填项。
	initial: 字段的初始值。
FileField:
	max_length: 文件名的最大长度。
	allow_empty_file: 是否允许上传空文件。
	required: 指定字段是否为必填项。
	initial: 字段的初始值。
ImageField:
	max_length: 图片文件名的最大长度。
	allow_empty_file: 是否允许上传空文件。
	required: 指定字段是否为必填项。
	initial: 字段的初始值。

上面每个Field类型中都有一个`error_messages`参数，为字典类型，key为max_length、required等值，value为该项校验不通过时的提示信息。

#### 字段校验器
同时Form模型中有一些常用的校验器：MinValueValidator、MaxValueValidator、RegexValidator、URLValidator、FileExtensionValidator等。

比如校验手机号是否合规：
```python
from django.core.validators import RegexValidator

class MyForm(forms.Form):
    phone_number = forms.CharField(validators=[RegexValidator(r'^\d{10}$', message='Please enter a valid phone number.')])
```

#### 自定义字段校验

在Django的Form中，`clean_<field>`方法用于对特定字段进行自定义验证和清洗操作。这些方法允许您在验证整个表单之前对单个字段进行特定的清洗和验证。

重写`clean_<field>`方法：
```python
from django import forms

class MyForm(forms.Form):
    name = forms.CharField()

    def clean_name(self):
        # 获取字段值
        name = self.cleaned_data.get('name')
        # 自定义验证逻辑 校验不通过抛出异常
        if len(name) < 3:
            raise forms.ValidationError("Name must be at least 3 characters long.")
		# 从数据库中查找是否存在
		
        # 返回清洗后的值
        return name
```

`clean`方法可以对多个字段进行校验，`clean`方法在调用表单的`is_valid()`方法时被执行，在表单的所有字段的`clean_<field>`方法被调用之后，`clean`方法被执行，如果所有字段的验证都通过了，那么`clean`方法将被调用。如果有字段的验证失败，`clean`方法不会被调用。

重写`clean`方法：
```python
from django import forms

class MyForm(forms.Form):
    start_date = forms.DateField()
    end_date = forms.DateField()

    def clean(self):
        cleaned_data = super().clean()
        start_date = cleaned_data.get("start_date")
        end_date = cleaned_data.get("end_date")

        # 表单级别的验证逻辑
        if start_date and end_date and start_date > end_date:
            raise forms.ValidationError("End date must be after start date.")

        # 清洗数据或其他逻辑
```

#### 提取错误信息

- `form.errors`：包含了html标签的错误信息。
- `form.errors.get_json_data()`：获取到的是一个字段类型的错误信息，key为字段名称，value为错误信息
- `form.errors.as_json()`：获取json格式的字符串。

#### ModelForm

有时表单中的字段和模型中的字段是一样的，而且表单中的数据也是模型中需要保存的数据，就可以将模型与表单进行绑定。

定义Model：
```python
from django.db import models

class MyModel(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField()
    age = models.IntegerField(MinValueValidator=(80, message="年龄不能超过80岁"))
```

定义Form，Form类继承FormModel：
```python
class MyModelForm(forms.ModelForm):
    class Meta:
        model = MyModel
        # fields = "__all__"  # 表示所有字段
        # exclude = ["create_time", "update_time"] # 排除需要校验的字段
        fields = ['name', 'email', 'age']
```

## cookie和session

`cookie`：http请求时无状态的，即某个用户已经访问过服务器了，第二次请求服务器仍然不知道当前请求是谁，cookie就是为了解决这个问题的。第一次请求的时候服务器会设置cookie，浏览器自动将cookie保存在本地，第二次请求的时候携带cookie，服务器就能知道当前请求是谁了。cookie存储量一般不超过`4kb`。

`session`：session的出现，是为了解决cookie存储数据不安全的问题。

web开发中涉及到cookie和session，成熟方案是：通过cookie存储一个`sessionid`，具体的数据保存在session中，请求服务器的时候携带cookie，服务器根据cookie中的`sessionid`从session库中查询session数据。django默认把session信息保存在数据库中，可以修改保存在redis中。

### cookie

设置cookie是给浏览器设置的，因此要通过`HttpResponse`对象设置，设置cookie的方法：`response.set_cookie()`，`set_cookie()`参数如下：

- key：cookie的key
- value：cookie的value
- max_age：最长生命周期，单位是秒
- expires：过期时间，是一个具体的日期。如果同时设置max_age和expires，将会使用expires的值作为过期时间
- path：对域名下哪个路径有效，默认是对所有域名都有效
- domain：对哪个域名有效，默认是针对域名下所有子域名都生效，如果要设置对某个子域名才有效，可以设置这个值
- secure：是否是安全的，如果设置为True，那么只能在https协议下才能使用
- httponly：默认是False，如果设置为True，那么在客户端不能通过JavaScript进行操作

```python
# 设置cookie
def add_cookie(request):
    response = HttpResponse("ok")
    response.set_cookie("username", "zhangsan", max_age=3600)
    return response

# 获取cookie
def get_cookie(request):
    # 获取单个
    username = request.COOKIES.get("username")
    print(f"username:{username}")

    # 获取所有cookie
    for key, value in request.COOKIES.items():
        print(key, value)
    return HttpResponse(username)

# 删除cookie
def delete_cookie(request):
    response = HttpResponse("ok")
    response.delete_cookie("username")
    return response
```

### session

通过`request.session`操作session，常用方法如下：

- get("key")：从session中获取指定值
- pop()：从session中删除一个值
- keys：从session中获取所有键
- items：从session中获取所有值
- clear：清除这个用户的session数据，清楚后sessionid及数据库中的数据还存在
- flush：删除session并且删除在cookie中存储的sessionid，一般在注销的时候用的比较多
- set_expire(value)：设置过期时间
	- 整形：代表秒数，多少秒后过期
	- 0：代表只要浏览器关闭，session就会过期
	- None：使用全局的session配置，在`setting.py`中可以设置`SESSION_COOKIE_AGE`来全局配置过期时间，默认是1209600秒，也就是两周的时间
- clear_expire：清除过期的session，django并不会清除过期的session，需要定期手动清理。

```python
# 设置session
def add_session(request):
    request.session["username"] = "zhangsan"
    return HttpResponse("ok")
```

???+ tip "注意"
    django中设值session，默认将session数据保存在数据库中（加密存储），同时在cookie中设值一条key为sessionid的数据，并不会在浏览器会话存储空间中设值数据。

## CSRF攻击

CSRF（Cross-Site Request Forgery），即跨站请求伪造，也被称为“One Click Attack”或者“Session Riding”，是一种网络攻击手段。它利用用户在浏览器中已登录的身份，伪装成用户发出的请求，对网站进行非法操作。CSRF攻击的原理是攻击者构造一个网站后台某个功能接口的请求地址，诱导用户去点击或者用特殊方法让该请求地址自动加载。由于用户在登录状态下，这个请求被服务端接收后会被误以为是用户合法的操作，从而导致攻击者实现对网站的非法操作。CSRF攻击对GET和POST形式的接口都存在威胁，尤其是GET形式的接口地址，攻击者可以轻易利用。

CSRF攻击的基本流程通常包括以下几个步骤：首先，用户登录到一个受信任的网站，并在本地生成Cookie信息。然后，用户在同一浏览器中打开另一个标签页，访问到攻击者控制的网站。这个网站包含有恶意请求，当用户浏览该网站时，恶意请求会被发送到受信任的网站。受信任的网站由于识别到用户的Cookie信息，会认为该请求是由用户自愿发起的，并以用户的权限处理该请求，导致恶意操作被执行。

CSRF攻击的危险性在于它不需要用户进行任何额外的操作，只需要用户已经登录到目标网站，并且Cookie信息没有过期。攻击者可以通过各种方式诱导用户访问包含恶意请求的网站，如发送带有恶意链接的电子邮件或社交媒体消息等。一旦用户点击这些链接，攻击就可以在不知情的情况下执行，给用户和网站带来潜在的风险。

为了防范CSRF攻击，可以采取多种措施，包括但不限于验证请求的来源地址、添加Token验证、二次验证、使用HTTPS协议、设置CORS策略等。通过这些措施，可以有效地提高Web应用的安全性，保护用户数据和资金安全。

Django框架提供了几种方法来防御CSRF攻击，主要包括：

1. 使用CSRF Token：Django通过在表单中添加一个CSRF token来防止CSRF攻击。这个token是随机生成的，并存储在服务器端的session中。当用户提交表单时，Django会验证表单中的token是否与服务器端session中的token相匹配。
2. 中间件启用：确保`django.middleware.csrf.CsrfViewMiddleware`在MIDDLEWARE配置中被激活，这个中间件负责在处理POST请求时检查CSRF token。
3. 模板标签：在HTML模板中使用{% csrf_token %}模板标签，这将生成一个隐藏的`<input>`字段，包含CSRF token的值。
4. AJAX请求：对于AJAX请求，需要在请求头中添加X-CSRFToken，其值应与cookie中的csrftoken值相匹配。
5. 视图装饰器：使用@csrf_protect装饰器来为特定视图添加CSRF保护，即使全局中间件被禁用。
6. 免除CSRF保护：在极少数情况下，如果需要为特定视图禁用CSRF保护，可以使用@csrf_exempt装饰器，但这种做法不推荐，因为它会带来安全风险。
7. 处理被拒绝的请求：Django允许通过CSRF_FAILURE_VIEW设置来指定一个视图，用于处理CSRF检查失败的情况。
8. 缓存与CSRF保护：如果使用缓存，应确保在需要插入CSRF token的视图上使用csrf_protect()装饰器，以避免缓存的响应中缺少CSRF保护。
9. 测试CSRF保护：Django的测试客户端可以设置为忽略CSRF检查，以便于开发和测试。
10. 边缘案例处理：对于不使用HTML表单或使用AJAX的页面，Django提供了sure_csrf_cookie()视图装饰器，以确保CSRF token cookie被设置。



## setttings
### 中间件配置

[中间件 | Django 文档 | Django](https://docs.djangoproject.com/zh-hans/5.0/topics/http/middleware/)

