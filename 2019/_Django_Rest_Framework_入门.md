我们这回来学习一下使用Django Rest Framework (DRF)(一个基于Django models，用来快速开发RESTFul API的应用), 为我们的Django Talk Project开发RESTFul API.

接着我们将会使用DRF将一个non-RESTFul 应用转换为RESTFul应用. 我们接下来将使用版本号2.4.2的DRF.

**这篇教程包含如下主题**:

1. DRF配置
2. RESTFul结构
3. 模型序列化(Model Serializer)
4. DRF Web Browseable API

如果你还没有看过前两个系列的教程，记得去温习一下。
1. [Django and AJAX Form Submissions – Say 'Goodbye' to the Page Refresh]('https://realpython.com/django-and-ajax-form-submissions/')
2. [Django and AJAX Form Submissions – More Practice]('https://realpython.com/django-and-ajax-form-submissions-more-practice/')

*此两篇文章待翻译。。。*

### DRF设置

安装：
```shell
pip install djangorestframework
pip freeze > requirements.txt
```
更新`settings.py`:
```python
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'talk',
    'rest_framework',
)
```

### RESTFul 结构

在一个RESTFul API中，节点(URLs)定义了API的结构以及用户如何通过HTTP方法(GET，POST，DELETE)成功的从我们的应用中获取数据。节点应该根据集合和元素符合逻辑的组织起来，集合和元素都是资源。在例子中，我们有一个简单的资源，`posts`，因此我们将使用如下的URLs - /posts/ 和 /posts/<id>，对应集合和特定的元素。

| GET           | POST           | PUT          | DELETE           |                  |
| ------------- | -------------- | ------------ | ---------------- | ---------------- |
| `/posts/`     | Show all posts | Add new post | Update all posts | Delete all posts |
| `/posts/<id>` | Show `<id>`    | N/A          | Update `<id>`    | Delete `id`      |

### DRF 快速开始

Reference: [Django Rest Framework – An Introduction]('https://realpython.com/django-rest-framework-quick-start/')
