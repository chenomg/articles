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

### RESTFul结构

在一个RESTFul API中，节点(URLs)定义了API的结构以及用户如何通过HTTP方法(GET，POST，DELETE)成功的从我们的应用中获取数据。节点应该根据集合和元素符合逻辑的组织起来，集合和元素都是资源。在例子中，我们有一个简单的资源，`posts`，因此我们将使用如下的URLs - /posts/ 和 /posts/<id>，对应集合和特定的元素。

| GET           | POST           | PUT          | DELETE           |                  |
| ------------- | -------------- | ------------ | ---------------- | ---------------- |
| `/posts/`     | Show all posts | Add new post | Update all posts | Delete all posts |
| `/posts/<id>` | Show `<id>`    | N/A          | Update `<id>`    | Delete `id`      |

### DRF快速开始

让我们开发新的API并运行吧

##### 模型序列化(Model Serializer)

DRF 的序列器将模型实例转化为Python 字典, 然后可以在转换为多种API中使用的格式 - 如JSON和XML. 和Django的ModelFormclass类似，DRF有一个简洁的序列器，ModelSerializerclass，非常容易使用：只要告诉它你想要模型中的哪些项：

```python
from rest_framework import serialziers
from talk.models import Post

class PostSerializer(serializers.ModelSerializer):

    class Meta:
        model = Post
        fields = ('id', 'author', 'text', 'created', 'updated')
```
将它保存在‘talk’目录中，文件名: `serializers.py`

##### 更新视图

我们需要更新现有的视图以适应RESTFul范例.更新当前视图如下：

```python
from django.shortcuts import render
from django.http import HttpResponse
from rest_framework.decorators import api_view
from rest_framework.response import Response
from talk.models import Post
from talk.serializers import PostSerializer
from talk.forms import PostForm


def home(request):
    tmpl_vars = {'form': PostForm()}
    return render(request, 'talk/index.html', tmpl_vars)

@api_view(['GET'])
def post_collection(request):
    if request.method == 'GET':
        posts = Post.objects.all()
        serializer = PostSerializer(posts, many=True)
        return Response(serializer.data)

@api_view(['GET'])
def post_element(request, pk):
    try:
        post = Post.objects.get(pk=pk)
    except Post.DoesNotExist:
        return HttpResponse(status=404)

    if request.method == 'GET':
        serializer = PostSerializer(post)
        return Response(serializer.data)
```
**这里发生了什么**：
1. 首先，装饰器@api_view检查传递进来的HTTP方法。现在，我们还只支持GET请求。
2. 然后，如果请求的是collection，则抓取全部数据，如果请求的是一个元素，则返回单个post
3. 最后，数据被序列化为JSON后返回

> 查看[官方文档]('http://www.django-rest-framework.org/api-guide/views#@api_view')以了解更多关于@api_view的资料

##### 更新URLs

接下来让我们设定新的URLs:

```python
# Talk urls
from django.conf.urls import patterns, url


urlpatterns = patterns(
    'talk.views',
    url(r'^$', 'home'),

    #api
    url(r'^api/v1/posts/$', 'post_collection'),
    url(r'^api/v1/posts/(?P<pk>[0-9]+)$', 'post_element')
)
```

##### 测试

现在让我们进行第一次测试！

1. 启动服务器，然后打开：[http://127.0.0.1:8000/api/v1/posts/?format=json]('http://127.0.0.1:8000/api/v1/posts/?format=json')
2. 现在我们学习一下如何使用[Browsable API]('http://www.django-rest-framework.org/topics/browsable-api').定位到[http://127.0.0.1:8000/api/v1/posts/]('http://127.0.0.1:8000/api/v1/posts/')
    干的漂亮，我们使用了这个很棒的，可阅读的API。
3. 获取一个元素是怎样呢？ 试一下：[http://127.0.0.1:8000/api/v1/posts/1]('http://127.0.0.1:8000/api/v1/posts/1')

在继续之前你应该已经注意到`author`字段是个`id`, 而不是实际的名字。我们接下来将会解释，现在我们使用新的API以使模板正常的工作。

### 重构REST

Reference: [Django Rest Framework – An Introduction]('https://realpython.com/django-rest-framework-quick-start/')
