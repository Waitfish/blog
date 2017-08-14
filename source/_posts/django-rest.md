---
title: 'django-restful 框架学习'
date: 2017-01-20 08:53:59
tags:
- django
- restful
- python
---


# Serialization 序列化
创建专门的序列化的 class 可以帮助 model 序列化
```python
class SnippetSerializer(serializers.ModelSerializer):
    class Meta:
        model = Snippet
        fields = ('id', 'title', 'code', 'linenos', 'language', 'style')
```
也可以自己创建序列化的函数,但是官方提供了modelserializer供我们继承


# request,response 对象
rest 框架重新封装了这2个对象.

## request
`request.post`可以得到客户端 post 的数据
`request.data`则可以处理 post\put\patch 等方法的数据(推荐使用data)

## response
`response`  对象是 `TemplateResponse`类型,它可以根据内容来决定返回给客户端的数据格式(比如:html 或者json)

```python
from format_suftrom django.conf.urls import url
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

urlpatterns = [
    url(r'^snippets/$', views.snippet_list),
        url(r'^snippets/(?P<pk>[0-9]+)$', views.snippet_detail),
        ]

urlpatterns = format_suffix_patterns(urlpatterns)
```
当客户端是浏览器的时候,就返回html代码,默认返回该格式
当客户端指定`endpoint`的时候就返回相应的格式.
`http://127.0.0.1/user.json` 就返回 json
`http://127.0.0.1/user.api` 就返回 html
客户端也可以在 `httpheader` 里面指定`accept`内容,比如`appication/json` `text/html`

<!-- more -->
# 使用 classview
和`django`一样,这个框架也有封装好的`classview`,具体的方法如下:
```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from django.http import Http404
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status


class SnippetList(APIView):
    ""
    List all snippets, or create a new snippet.
    ""
    def get(self, request, format=None):
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    def post(self, request, format=None):
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)"
```
## 还有高级的混合类
```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework import mixins
from rest_framework import generics

class SnippetList(mixins.ListModelMixin,
    mixins.CreateModelMixin,
    generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
```
抽象了常见的`list\create`等方法

## 最高级的classbase类
```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework import generics


class SnippetList(generics.ListCreateAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer


class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```
使用这个高级的`view`类,两行代码就完成了 `get\put\delete`方法,全都浓缩了.

## 使用基于类的 url 配置
```python
from django.conf.urls import url
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

urlpatterns = [
    url(r'^snippets/$', views.SnippetList.as_view()),
    url(r'^snippets/(?P<pk>[0-9]+)/$', views.SnippetDetail.as_view()),
        ]

urlpatterns = format_suffix_patterns(urlpatterns)
```
# 认证和权限控制
## 对`model`进行权限控制的时候,需要在`model`加上`django`的用户外键

```python
owner = models.ForeignKey('auth.User', related_name='snippets', on_delete=models.CASCADE)
```
外键也需要提供默认值,键值是用户`id`
```python
owner = models.ForeignKey('auth.User', related_name='snippets', on_delete=models.CASCADE,defaut=key)
```
访问用户的时候可以看到属于用户的对象信息需要
添加用户的序列号文件

```python
from django.contrib.auth.models import User

class UserSerializer(serializers.ModelSerializer):
    snippets = serializers.PrimaryKeyRelatedField(many=True, queryset=Snippet.objects.all())

    class Meta:
        model = User
        fields = ('id', 'username', 'snippets')
```

## 关联用户和模块 perform_create

在对应的`model`的`view`类中,添加`perform_create`方法
```python
def perform_create(self, serializer):
    serializer.save(owner=self.request.user)
```

## 在model的序列化类中添加 用户的信息

```python
owner = serializers.ReadOnlyField(source='owner.username')
```
`source`可以指定需要输出的属性

## 基本的权限控制

从框架中导入默认的权限控制
```python
from rest_framework import permissions
```
在对应的view中添加该属性即可实现基本的权限控制
```python
permission_classes = (permissions.IsAuthenticatedOrReadOnly,)
```

## 定制的权限控制
可以自建一个权限控制的文件,用来控制
```python
from rest_framework import permissions


class IsOwnerOrReadOnly(permissions.BasePermission):
    ""
    Custom permission to only allow owners of an object to edit it.
    ""

    def has_object_permission(self, request, view, obj):
        ""
        Read permissions are allowed to any request,
        so we'll always allow GET, HEAD or OPTIONS requests.
        ""
        if request.method in permissions.SAFE_METHODS:
            return True

        return obj.owner == request.user`
```
```python
permission_classes = (permissions.IsAuthenticatedOrReadOnly,
                      IsOwnerOrReadOnly,)
```
# 给api加上超级链接
## 修改模块的序列化类

在序列化的类上继承 `HyperlinkedModelSerializer`,并使用`HyperlinkedRelatedField`

```python
class UserSerializer(serializers.HyperlinkedModelSerializer):
    snippets = serializers.HyperlinkedRelatedField(many=True, view_name='snippet-detail', read_only=True)

    class Meta:
        model = User
        fields = ('url', 'id', 'username', 'snippets')
```
__需要在 url 配置中添加 name 属性__

## 列表页面的分页配置
在项目的`setting.py`文件中添加下面的属性
```python
REST_FRAMEWORK = {
        'PAGE_SIZE': 10
        }
```
# viewset 和 router
## 终极抽象 view 类,viewset
和单独的 `view` 不同的是,`viewset`抽象了一般的`list`和`detail`方法(list\create\retrieve\update\destroy)

```python
from rest_framework.decorators import detail_route

class SnippetViewSet(viewsets.ModelViewSet):
    ""
    This viewset automatically provides `list`, `create`, `retrieve`,
    `update` and `destroy` actions.

    Additionally we also provide an extra `highlight` action.
    ""
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
    permission_classes = (permissions.IsAuthenticatedOrReadOnly,
    IsOwnerOrReadOnly,)

    @detail_route(renderer_classes=[renderers.StaticHTMLRenderer])
    def highlight(self, request, *args, **kwargs):
        snippet = self.get_object()
        return Response(snippet.highlighted)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)"
```

## 抽象的 router


```python
from django.conf.urls import url, include
from snippets import views
from rest_framework.routers import DefaultRouter

# Create a router and register our viewsets with it.
router = DefaultRouter()
router.register(r'snippets', views.SnippetViewSet)
router.register(r'users', views.UserViewSet)

# The API URLs are now determined automatically by the router.
# Additionally, we include the login URLs for the browsable API.
urlpatterns = [
    url(r'^', include(router.urls)),
    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework'))
        ]
```
# 添加 schema
安装`coreapi`
`pip install coreapi`
```python
from rest_framework.schemas import get_schema_view

schema_view = get_schema_view(title='Pastebin API')

urlpatterns = [
    url('^schema/$', schema_view),
        ...
        ]
```
# 官方示例项目地址
github:https://github.com/tomchristie/rest-framework-tutorial

demo  :http://restframework.herokuapp.com/snippets/
