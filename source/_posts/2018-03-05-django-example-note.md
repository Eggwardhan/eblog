---
layout: post
title: django note
tag: django
---
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

添加如下代码到views.py文件中：

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

这是一个帖子详情视图（view）。这个视图（view）使用year，month，day以及post作为参数通过给予slug和日期来获取到一篇已经发布的帖子。请注意，当我们创建Post模型（model）的时候，我们给slgu字段添加了unique_for_date参数。这样我们可以确保在给予的日期中只有一个帖子会带有一个slug，因此，我们能通过日期和slug取回单独的帖子。在这个详情视图（view）中，我们通过使用get_object_or_404()快捷方法来检索期望的Post。这个函数能取回匹配给予的参数的对象，或者当没有匹配的对象时返回一个HTTP 404（Not found）异常。最后，我们使用render()快捷方法来使用一个模板（template）去渲染取回的帖子。
