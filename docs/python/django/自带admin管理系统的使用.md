## setting.py

1、`INSTALLED_APPS`配置了`django.contrib.admin`
2、`LANGUAGE_CODE`设置为`zh-hans`，将admin界面语言改为中文

## 定义模型

```python
class BlogCategory(models.Model):
    name = models.CharField(max_length=100)

    def __str__(self):
        return self.name

    class Meta:
	    # verbose_name用于在admin中显示，verbose_name_plural用于在admin中显示复数形式
        verbose_name = '博客分类'
        verbose_name_plural = verbose_name
```

## 将模型注册到admin

```python title="admin.py"
from django.contrib import admin
from .models import *


class BlogCategoryAdmin(admin.ModelAdmin):
    # admin管理 列表页 显示的字段
    list_display = ['name']


class BlogAdmin(admin.ModelAdmin):
    # admin管理 列表页 显示的字段
    list_display = ['title', 'content', 'pub_time', 'category', 'author']


class BlogCommentAdmin(admin.ModelAdmin):
    list_display = ['content', 'pub_time', 'author', 'blog']


# 注册模型到admin
admin.site.register(BlogCategory, admin.ModelAdmin)
admin.site.register(Blog, BlogAdmin)
admin.site.register(BlogComment, BlogCommentAdmin)
```

## auth_user表

`auth_user`表中`is_superuser`设置为`1`表示超级用户，拥有所有权限；`is_staff`为`1`的用户为员工，才能登录到admin管理后台。

