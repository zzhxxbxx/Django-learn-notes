@[TOC](Django:url的应用)

# Django:url的应用

## url映射
1. 为什么会去urls.py中寻找映射？   
 `settings.py`中配置了`ROOT_URLCONF = 'xxbxx.urls'`。所有django会去`urls.py`中寻找
2. `urls.py`中获取所有映射，都应该放在`urlpatterns=[]`中。
3. 所有映射需要用`path`函数或者`re_path`函数进行包装。注：`re_path`支持正则。
## url传参
1. 采用url中使用变量的方式：   
    在path的第一个参数中，使用`<参数名>`的方式可以传递参数，然后在视图函数中也要写一个参数，
    视图函数中的参数名必须和url中的参数名保持一致。   
    另外，url中可以传递多个参数。
2. url参数转换器:
    1. str：除了`/`以外所有的字符都是可以的     
    2. int：一个或多个阿拉伯数字    
    3. path：所有的字符都满足 
    4. uuid：满足`uuid.uuid4()`这个函数返回的字符串的格式    
    5. slug：英文横杠/英文字符/阿拉伯数字/下划线
    
3. 采用查询字符串的方式：
    在url中，不需要单独匹配查询字符串的部分，只需要在视图函数中使用`request.GET.get('参数名‘)`的方式来获取参数的值。
    ```python
   def get_id(request):
       n = request.GET.get('id')
       return n
    ```
    因为查询字符串使用的是`get`请求，所以通过`request.GET`来获取参数，
    并且是因为`GET`是个类似于字典的数据类型，所以获取值的方式和字典的方式都是一样的

##url模块化：   
如果项目越来越大，那么url变得越来越多，如果都放在`urls.py`文件中，那末将不太好进行管理。
因此，我们将每个app的url放在自己的urls中进行管理。
一般我们会在app中新建一个`urls.py`文件，用来存储和管理该app的url。  
1. 项目的`urls.py`应用`include`函数包含子`urls.py`
    ```python
   from django.contrib import admin
   from django.urls import path, include
   
   urlpatterns = [
       path('admin/', admin.site.urls),
       path("book/", include('xxbxxUrl.urls')),
   ]
    ```
2. 在app的`urls.py`,所有的url匹配也要方一个叫做`urlpatterns`的变量，否则会404。
3. `url`要根据主`urls.py`和app中的`urls.py`进行拼接的，因此注意不要多加`/`。
## url命名
1. 为什么需要url命名？
    因为url经常变化，如果代码中写四可能会经常改动代码。
    给url取名字，以后使用时用他的名字反转就可以得到url了。
2. 如何给一个url指定名称？
    在`path`函数中，传递一个`name`的参数
    ```python
   from django.urls import path, include
   from django.contrib import admin
   path('admin/', admin.site.urls, name="admin")
    ```
    通过name反转获取url：
    ```python
   from django.http import HttpResponse
   from django.shortcuts import redirect, reverse
   def index(request):
       return redirect(reverse("url_name"))
    ```
3. 应用(app)命名空间
    在多个app之间，有可能产生同名url。这时候可以使用应用命名空间，来做区分。     
    在app的`urls.py`中配置`app_name=’app_name‘`属性。   
    访问路径`url='$[app_name]:$[url_name]`
4. 实例命名空间
    一个app，可以创建多个实例。
    可以使用多个url映射用一个app。
    在做反射的时候，如果使用命名空间就可能发生混淆。
    为此引入了实例命名空间
    应用：在要在`include`函数中传递一个`namespace`参数即可
    ```python
   from django.urls import path, include
   path("book/", include('xxbxxUrl.urls', namespace='book')) 
    ```
    在做反转时，根据实例命名空间来制定具体的url：
    ```python
   from django.shortcuts import redirect, reverse
   def index(request):
       # 获取实例命名空间
       current_namespace = request.resolver_math.namespce
       return redirect(reverse('%s:url_name' % current_namespace))
    ```
    `注：指定namespace之前必须指定app_namespace`

## 函数补充
### include函数
1. include(module, namespace=None):
    * module：子url的模块字符串
    * namespace：实例命名空间
2. include((pattern_list, app_namespace), namespace=None):
3. include(pattern_list, namespace=None): 
4. `等待详细补充`
### re_path函数
1. `re_path`和`path`的作用是一样的，只不过`re_path`在写path的时候可以使用正则表达式，功能更加强大。
2. 写正则表达式推荐使用原生字符串，也就是以`r`开头的字符串。
3. 在正则表达式中定义变量，需要使用`()`阔起来。这个参数是由名字的，那么使用`?p<参数的名字>`，然后在后面添加正则表达式的规则。
    ```python
   from django.urls import re_path, include
   re_path(r"^list/(?p<year>\d{4})/$", include("xxbxxUrl.urls"))
    ```        
4. 如果不是特别要求，直接用`path`就够了，除非是url中确定是需要使用正则表达式来解决才使用`re_path`;
### reverse函数
1. 如果在反转url的时候，需要传递参数，那么可以使用`kwargs`参数到`revers`函数中：
    ```python
   from django.shortcuts import redirect, reverse
   detail_url = reverse('url_name', kwargs={'key1':'value1', 'key2':"value2"})
    ```
2. 如果需要查询字符串的参数，则需要手动进行拼接：
    ```python
   from django.shortcuts import reverse
   detail_url = reverse("url_name") + "?key=value" 
    ```
### 自定义url转换器
django为我们提供一些内置的url转换器，包括int，uuid等。
同时我们也可以自定义url转换器
自定义转换器的五个步骤：
   1. 定义一个类，直接继承object就可以了。
   2. 在类中定义一个属性regex，属性值为限制url的正则表达式。
   3. 实现to_python方法，将url中的值转一下，然后返回给视图。
   4. 实现to_url方法，在做url反转的时候将传进来的参数转换后拼接成正确的url。
   5. 将定义好的转换器，注册到django中，使用`django.urls.converters.register_converter`。
       ```python
      from django.urls import converters, register_converter
      class MyselfConverter(object):
          regex = r'\w+\w'   
      
       
          def to_python(self, value):
              result = value.split("+")
              return result
          
          
          def to_url(self, value):
              if isinstance(value, list):
                  return "+".join(list)
              else:
                  raise RuntimeError("转换为url失败！")
      register_converter(MyselfConverter, "myself")
       ``` 
   6. 规范：
       1. 在app下创建`converter.py`文件，来存放自定义url转换器。
       2. 在`__init__.py`下引用这个类`from . import converter`，便于将自定义的url转换器注册到django中。
