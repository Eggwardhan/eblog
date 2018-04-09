---
layout: post
title: django note
tag: django
---

## Tips  --　creat an independence Py environment
I strongly recommend you using **virtualenv** .In this way  ,you can use different version of package to correspond to different program

```

pip install virtualenv
virtualenv my_env
//how to locate ｐｙ３
zenx\$ *which python3*
/Library/Frameworks/Python.framework/Versions/3.5/bin/python3
zenx\$ *virtualenv my_env -p
/Library/Frameworks/Python.framework/Versions/3.5/bin/python3*
source my_env/bin/activate  //activate your virtualenv  ,and use **deactive** to stop.
         //Along with *virtualenvwrapper* tool,you can enjoy more conveniencec.

```
### QuerySet and managers

```
>>> from django.contrib.auth.models import User
>>> from blog.models import Post
>>> user = User.objects.get(username='admin')  #get方法允许你从数据库中取回唯一对象
>>> post = Post.objects.create(title='One more post',
                        slug='one-more-post',
                        body='Post body.',
                        author=user)
>>> post.save()
```
ＯＲＭ是基于queryset的。QuerySet是从你的数据中按条件采集对象。每个Ｄｊａｎｇｏ　models　至少有一个manager,默认叫做objects.若要获取每一个对象

\>>>　all_posts= Post.objects.all()

---

### filter()
过滤查询集的方法　

\>>>Post.objects.filter(publish__year=2018)

或者多个字段过滤=
```
>>>Post,objects.filter(publish__year=2018,author__username='admin')
或者

>>>Post,objects.filter(publish__year=2018).filter(author__username='admin')
```

### exclude()
排除　　方法同上
### order_by(  )
Post.objects.order_by('title')# 默认升序，可加负号来指定降序
### delete()
获取对象后使用

---

### 添加模型管理器
1. 添加额外 manager,  如 Post.object.my_manager()
2.　继承 manager , Post.my_manager.all()

```
class PublishedManager(models.Manager):
  def get_queryset(self):
    return super(PublishedManager,
                 self).get_queryset().filter(status='published')

class Post(models.Model):
    #...
    objects = models.Manager() #the default managers
    published = PublishedManager() #Our custom manager                 
```
### 构建列和详情视图(views)
一个ｄｊａｎｇｏ视图就是一个ｐｙ方法，它可以接受一个ｗｅｂ请求然后返回一个web响应。在ｖｉews中通过所有的逻辑处理返回期望的响应。

创建views然后为每个视图定义一个Ｕｒｌ模式，我们将会创建ＨＴＭＬ模板来渲染这些视图生成的数据。每一个视图都会渲染模板传递变量给它然后会返回一个经渲染输出的ＨＴＴＰ响应。

编辑应用下的views.py

```

from django.shortcuts import render,get_object_or_404
from .models import post
def post_list(request):
    posts =Post.published.all()
    return render(request,
      'blog/post/list.html',
      {'posts':posts})
```

post_list视图将request对象作为唯一参数。！所有的视图都需要有这个参数。获我们先获取了所有状态为已发布的帖子，通过之前创建的 published manager.
最后　，我们使用ｄｊａｎｇｏ提供的render()通过给予的模板(template)来渲染帖子列
这个函数将request对象作为参数，模板（template）路径以及变量来渲染的给予的模板（template）。它返回一个渲染文本（一般是HTML代码）HttpResponse对象。render()方法考虑到了请求内容，这样任何模板（template）内容处理器设置的变量都可以带入给予的模板（template）中。

添加如下代码到views.py文件中来展示单独的帖子：

```

def post_detail(request, year, month, day, post):
    post = get_object_or_404(Post, slug=post,
                                   status='published',
                                   publish__year=year,
                                   publish__month=month,
                                   publish__day=day)
    return render(request,
                  'blog/post/detail.html',
                  {'post': post})

```

这是一个帖子详情视图（view）。这个视图（view）使用year，month，day以及post作为参数通过给予slug和日期来获取到一篇已经发布的帖子。*请注意*，当我们创建Post模型（model）的时候，我们给slgu字段添加了unique_for_date参数。这样我们可以确保在给予的日期中只有一个帖子会带有一个slug，因此，我们能通过日期和slug取回单独的帖子。



在这个详情视图（view）中，我们通过使用get_object_or_404()快捷方法来检索期望的Post。这个函数能取回匹配给予的参数的对象，或者当没有匹配的对象时返回一个HTTP 404（Not found）异常。最后，我们使用render()快捷方法来使用一个模板（template）去渲染取回的帖子。
### Add URL patterns to your views
一个url模式是由一个ｐｙ正则表达式一个ｖｉｅｗ一个全项目范围内的命名组成，Django在运行中会遍历所有ＵＲＬ模式直到地一个匹配请求ＵＲＬ才停止。之后导入匹配的URL模式的view并执行
在blog应用下创建urls.py
```

from django.conf.urls import url
from . import views
urlpatterns = [
    # post views
    url(r'^$', views.post_list, name='post_list'),
    url(r'^(?P<year>\d{4})/(?P<month>\d{2})/(?P<day>\d{2})/(?P<post>[-\w]+)/$',
        views.post_detail,
        name='post_detail'),
]

```
第一条Url无参，它映射到post_list view。第二条带上４个参数映射。

将blog中的ｕｒｌ模式包含到主项目的主url模式中编辑你的项目中的ｍｙｓｉｔｅ文件夹中的urls.py文件

```

from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),
    url(r'^blog/', include('blog.urls',
        namespace='blog',
        app_name='blog')),
]


```

#### add templates to views

```

templates/
    blog/
        base.html
        post/
            list.html
            detail.html


```

创建模板目录结构如上　base.html文件会包含站点主要的HTML结构以及分割内容区域和一个导航栏。list.html detail.html会继承base.html文件来渲染各自的blog帖子列和view
django 有一个强大的templates语言允许你指定数据的如何进行展示。基于模板标签，如{%tag%},{{variable}}以及templates filters,可以对变量进行过滤,如{{variable|filter}}。你可以访问django documentation ref/templates/builtins/找到内置的所有templates tags and filters.
添加base.html文件并添加代码

```

{% load staticfiles %}   //加载django.contrib.staticfiles应用提供的staticfiles模板标签　，then　你可以在template中使用{%static%}模板过滤器－》包含一些静态文件如css。
<!DOCTYPE html>
<html>
<head>
  <title>{% block title %}{% endblock %}</title>
  <link href="{% static "css/blog.css" %}" rel="stylesheet">
</head>
<body>
  <div id="content">
    {% block content %}　　　　//{%block%}标签　定义一个区块继承模板
    {% endblock %}
  </div>
  <div id="sidebar">
    <h2>My blog</h2>
      <p>This is my blog.</p>
  </div>
</body>
</html>

```      

编辑post/list.html文件

```
{% extends "blog/base.html" %} //继承base.html的template　然后在title 和content　block　中填充内容

{% block title %}My Blog{% endblock %}

{% block content %}
  <h1>My Blog</h1>
  {% for post in posts %}　　　//循环迭代来展示内容等
    <h2>
      <a href="{{ post.get_absolute_url }}">
        {{ post.title }}
      </a>
    </h2>
    <p class="date">
      Published {{ post.publish }} by {{ post.author }}
    </p>
    {{ post.body|truncatewords:30|linebreaks }}　　　　//filters １号truncatewords用来限制内容缩短于一定的字数内　　２号linebreaks转换内容中的换行符为HTML的换行符
  {% endfor %}
{% endblock %}
```
然后是detail.html文件

```
{% extends "blog/base.html" %}

{% block title %}{{ post.title }}{% endblock %}

{% block content %}
  <h1>{{ post.title }}</h1>
  <p class="date">
    Published {{ post.publish }} by {{ post.author }}
  </p>
  {{ post.body|linebreaks }}
{% endblock %}

```
#### 添加页码

```
from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger

def post_list(request):
    object_list = Post.published.all()
    paginator = Paginator(object_list, 3) # 3 posts in each page
    page = request.GET.get('page')
    try:
        posts = paginator.page(page)
    except PageNotAnInteger:
        # If page is not an integer deliver the first page
        posts = paginator.page(1)
    except EmptyPage:
        # If page is out of range deliver last page of results
        posts = paginator.page(paginator.num_pages)
    return render(request,
                  'blog/post/list.html',
                  {'page': page,
                   'posts': posts})
```
How  Paginator works:
- 希在每页中显示的对象数量来实例化Paginator类
- 获取到page GET参数来指明页数
- 通过调用page()方法在期望的页面中获得了对象。
- 如果page获得的参数不是一个整数　，我们就返回第一页的结果。如果这个参数数字超出了最大的页数，我们就展示最后一页的结果
- 我们传递页数并获取对象给 template

创建template来展示分页处理，它可以被任意的模板包含来使用分页。在blog　应用的templates文件夹下创建一个新文件 pagination.html　
```
<div class="pagination">
  <span class="step-links">
    {% if page.has_previous %}
      <a href="?page={{ page.previous_page_number }}">Previous</a>
    {% endif %}
    <span class="current">
      Page {{ page.number }} of {{ page.paginator.num_pages }}.
    </span>
      {% if page.has_next %}
        <a href="?page={{ page.next_page_number }}">Next</a>
      {% endif %}
  </span>
</div>
```
为了渲染上一页与下一页的链接并且展示当前页面和所有页面的结果，这个分页模板（template）期望一个Page对象。让我们回到blog/post/list.html模板（tempalte）中将pagination.html模板（template）包含在{% content %}区块（block）中，传递给模板的Ｐａｇｅ对象叫做posts，我们将分页模板包含在帖子列模板中指定参数来对它进行正确的渲染。这种方法你可以反复使用，用你的分页模板对同的models views进行分页处理如下所示：
```
{% block content %}
  ...
  {% include "pagination.html" with page=posts %}
{% endblock %}
```

### 基于类的views
__因为一个视图view的调用就是得到一个web请求并且返回一个web响应__，你可以将你的view定义成类方法。DJango为此定义了基础的view类。他们都url从view类继承而来，view类可以操控http方法调度以及其他功能。这是一个可替代的方法来创建你的views.
我们准备通过Django提供的通用ListView使我们的Post_list　view转变为一个基于类的视图。这个基础view允许你　对任意的对象进行排列。
编辑blog应用下的view.py
```
from django.views.generic import ListView
class PostListView(ListView):
     queryset = Post.published.all()   #特定的查询集(QuerySet)代替取回所有的对象。代替定义一个queryset属性，我们可以指定model =Post然后Django将会构建Post.object.all()查询集(queryset)给我们
     context_object_name ='posts'　　#使用环境变量posts给查询结果。如果我们不指定任意的context_object_name默认的变量将会是object_list.
     paginate_by = 3　　　　　　　　#对结果分页处理每页只显示３对象
     template_name = 'blog/post/list.html'　　　#使用定制的template来渲染页面　如果不设置默认的将会使用blog/post_list.html

```
打开blog应用下的urls.py注释到之前的post_listURL模式，在在之后添加一个新的URL模式来使用PostlistView类
```
urlpatterns =[
      #post views
      #url(r'^$',views.post_list,name='post_list'),
      url(r'^$',views.PostListView.as_view(),name='post_list'),
      url(r'^(?<year>\d{4})/(?P<month>\d{2})/(?P<day>\d{2})/'\
          r'(?P<post>[-\w]+)/$',
          views.post_detail,
          name='post_detail'),
]
```
为了保持分页处理能工作，我们必须将正确的页面传递给template。Django的ListView通过叫做Page_obj的变量来传递被选择的页面，所以你必须编辑你的Post_list_html template去包含使用了正确的变量的分页处理
```
{% include "pagination.html" with page =page_obj %}
```
