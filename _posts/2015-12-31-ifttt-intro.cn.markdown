---
layout: post
title:  "IFTTT：沟通应用的桥梁"
description: 向在国内的朋友介绍一个好玩的东西：IFTTT
date:   2015-12-31 21:31:20
categories: technology
---

### 序言
在2015年的尾牙，琳琅满目的社交平台已经成为了我们生活中不可或缺的一部分。可以说，我们的生活状态有相当一部分都是通过各种互联网平台表现与传播的。它们同时也反映了我的自我认知和外在形象。盛世之下，各种相关应用已是呈井喷之态。

其中，有偏向于搭建公共、开放平台的Twitter和微博，有基于熟人小圈子互动的微信和Snapchat，也有介于两者之间的Instagram和Facebook。其中Facebook又更偏重于朋友间的常规联络和分享，以及适量的在公共主页和热点事件上的互动。相比之下，Instagram则是一个全面封闭的侧重于营销和做个人品牌推广（branding）的平台（特别是对于摄影师，设计师，模特等而言）。在这些巨人之下，还有着千千万万的“小”公司占据着各种细分市场，比如陌陌，抱抱，17……

如此之多的应用共同编织出了我们丰富的线上、线下生活，服务了我们的方方面面。那么，有没有什么办法能够将它们连接起来，让我们的生活更智能呢？

### IFTTT是什么？
今天的主角终于出场了。IFTTT，一家总部位于旧金山的新创公司，致力于让人们发挥创造力，去控制自己喜欢的产品和应用。让我们想象一下，假如我们在更新微博状态的时候，想同时将这条状态分享到Facebook上（自动翻墙了？），假如我们想在朋友生日的时候自动通过Facebook或者微博进行问候，假如我们想让我们的Facebook和Twitter里面的个人信息保持同步，等等。我们应该怎么做呢？IFTTT为我们提供了一个完全自动化的解决方案。其实，IFTTT的名字来源于其功能：<strong style="color:#3BB9FF;">if this, then that</strong>。对于之前的情景，我们发现全部都可以套进这句话当中：if有新的微博状态发布，then将其分享到Facebook。if年／月／日，then发送Facebook或微博消息。if Facebook用户信息变动，then在Twitter账户中进行相同的设置。只要我们想要连接的应用在IFTTT的平台上推出了自己的服务，那么我们就可以根据它们提供的服务种类，向搭乐高积木一样构建出既有创意，又方便实用的功能。

### 生态系统及功能详述
从成立到现在短短几年，IFTTT平台上已经有了上百个频道。在其术语当中，频道（Channel）即对应一个应用，或是说产品。比如微博是一个频道，Facebook是另外一个频道。
<div class='md-10-suffix-1'>
  <figure>
    <img src="/assets/img/post-ifttt-intro/channels.png" alt="IFTTT上的应用"/>
    <figcaption>IFTTT上的应用</figcaption>
  </figure>
</div>

国外大多数比较流行的应用都已经在上面有了自己的频道，并且支持相当多的功能。但不得不说，国内的公司在其平台上的存在感还很低，目前我只看到了微博，并且仅提供简单的功能接口，比如监测到新发微博，发送微博等。

我们已经说到，IFTTT这个平台，由上百个频道组成。而根据其对平台功能的抽象（<strong style="color:#3BB9FF;">if this, then that</strong>模型），一个频道又由**Triggers**和**Actions**组成。其中，Trigger是一个频道可以为其他频道提供的信息。
<div class='md-10-suffix-1'>
  <figure>
    <img src="/assets/img/post-ifttt-dev/triggers.png" alt="Facebook Triggers"/>
    <figcaption>Facebook Triggers</figcaption>
  </figure>
</div>

上图是Facebook可以提供的部分Trigger，即是 <strong style="color:#3BB9FF;">if this, then that</strong> 这个模型中的**this**部分。

Action则是一个频道接到其控制频道（提供Trigger）的信息之后，可以采取的操作。

<div class='md-10-suffix-1'>
  <figure>
    <img src="/assets/img/post-ifttt-dev/actions.png" alt="Twitter Actions"/>
    <figcaption>Twitter Actions</figcaption>
  </figure>
</div>

上图是Twitter支持的部分Action。所以，当我们把这两个频道连接在一起的时候，我们先在Facebook上选择一个Trigger，比如“New status message by you”，然后再在Twitter上选择一个Action，比如“Post a tweet”。这样，每当我们在Facebook上更新一条状体，这条状态就会自动被发布到我们的Twitter上面。最终组装完成的模型就是”if New status mssage by you, then Post a tweet”。简单易用。

需要说明的是，一个频道可以既有Trigger，又有Action。在上面的例子中，我们完全可以将Twitter和Facebook调换位置，让我们在Twitter上的行为去驱动Facebook。提供什么样的Trigger和Action给用户，取决于频道的开发团队。

### 更多的创意
虽然我在序言和后面的说明中都是以社交类App作为例子，但是IFTTT并不局限于此。任何一个产品都可以在其平台中搭建自己的频道，包括智能硬件（须有服务器端）。在IFTTT里，两个应用组成的连接，被称为一个Recipe。在它们的网站中，已经有了成千上万的Recipe。这其中大部分都来自用户。当用户自己搭建了一个Recipe之后，如果他觉得很酷，就可以将其发布出去，供他人使用。这种基于UG(User-generated Content)的方式，对于保持平台的活力也非常有帮助。下面简单给大家截取一些Recipe。

<div class='md-10-suffix-1'>
  <figure>
    <img src="/assets/img/post-ifttt-intro/recipes1.png" alt="Smart Home Recipes"/>
    <figcaption>智能家居的Recipes</figcaption>
  </figure>
</div>

<div class='md-10-suffix-1'>
  <figure>
    <img src="/assets/img/post-ifttt-intro/recipes2.png" alt="Android Recipes"/>
    <figcaption>安卓手机的Recipes</figcaption>
  </figure>
</div>

### 工作原理
在这样一个架构当中，IFTTT充当着一个中间人的角色。当一个频道上线之后，IFTTT的服务器会每隔15分钟去访问一个Trigger的节点，如果得到新数据，则自动向这个Trigger所连接的Action节点发送一条指令，以驱动一个操作。当然实际情况要稍微复杂一点，比如用户可以向Trigger提供输入、另一类只有Action的Recipe（称为“Do Recipe”）和更加实时的解决方案等等。具体的原理和开发指南可以参见我的另一篇文章：[Channel Development on IFTTT]。在了解了上述原理之后，想必大家已经发现，只要IFTTT不被墙，那么我们还可以将它作为桥梁去控制墙外的应用。赶快[体验]一下吧。



[Channel Development on IFTTT]: {% post_url 2015-12-25-ifttt-development %}
[体验]:                          https://ifttt.com/recipes
