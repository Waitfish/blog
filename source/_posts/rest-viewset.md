---
title: django-restful 的 viewset
date: 2017-01-21 10:28:35
tags:
- python
- django
- restful
---

# ViewSet
是框架的基类,没有提供`list` `create`等方法,继承之后可以自定义每个方法.

```python
class UserViewSet(viewsets.ViewSet):
    ""
    Example empty viewset demonstrating the standard
    actions that will be handled by a router class.

    If you're using format suffixes, make sure to also include
    the `format=None` keyword argument for each action.
    ""

    def list(self, request):
        pass

    def create(self, request):
        pass

    def retrieve(self, request, pk=None):
        pass

    def update(self, request, pk=None):
        pass

    def partial_update(self, request, pk=None):
        pass

    def destroy(self, request, pk=None):
        pass"
```

# ModelViewSet
模板基类,提供了所有框架有的方法

```python
class UserViewSet(viewsets.ModelViewSet):
    ""
    A viewset that provides the standard actions
    ""
    queryset = User.objects.all()
    serializer_class = UserSerializer

    @detail_route(methods=['post'])
    def set_password(self, request, pk=None):
        pass

    @list_route()
    def recent_users(self, request):
        pass
```
如果要定制某些方法,可以使用修饰符,比如`@list_route()`,还可以接收其他的参数

```python
@detail_route(methods=['post'], permission_classes=[IsAdminOrIsSelf])
def set_password(self, request, pk=None):
    ...
```
使用定制方法后的`url`如下
```
^users/{pk}/unset_password/$
```
# ReadOnlyModelViewSet
提供常见的`get` `list`方法
# 定制自己的基类
混合`Mixin`
```python
class CreateListRetrieveViewSet(mixins.CreateModelMixin,
    mixins.ListModelMixin,
    mixins.RetrieveModelMixin,
    viewsets.GenericViewSet):
    ""
    A viewset that provides `retrieve`, `create`, and `list` actions.

    To use it, override the class and set the `.queryset` and
    `.serializer_class` attributes.
    ""
    pass
```
