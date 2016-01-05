---
layout: post
title:  "Channel Development on IFTTT"
description: Share my experience of developing on IFTTT since the past half year.
date:   2015-12-25 16:31:20
categories: technology
---

<h3> What is IFTTT? </h3>
<p> Suppose every time you update your status on Facebook, you also want to post exactly the same content to your Twitter
  account. And every time you upload a photo to Instagram, you want it to be saved to your Dropbox as well.
  How would you do it automatically? Here is when IFTTT comes in. It is short for
  "if<strong style="color:#3BB9FF;"> this</strong>, then <strong style="color:#3BB9FF;">that</strong>".
  Fairly intuitive. You can put whatever you want in <strong style="color:#3BB9FF;"> this</strong> and
  <strong style="color:#3BB9FF;"> that</strong>.<p>


<h4> Channels, Triggers and Actions </h4>
<p> Channels are Twitter, Facebook, email, Google Drive etc. Triggers are what can be used to prompt an action,
  such as "New status message", "New photo post", and "You are tagged in a photo" from Facebook. Action is the
  work that IFTTT undertakes as a result of a Trigger. Examples are "Send me an email" from the email channel,
  "Create a status message" from Facebook, and even "Update device wallpaper" from the Android device channel. </p>

<p> Now, to get our first job done, we only need "if <strong style="color:#3BB9FF;">New status message</strong>,
  then <strong style="color:#3BB9FF;">Post a tweet</strong>." The former is a <strong>Trigger</strong> provided by
  Facebook, while the latter is an <strong>Action</strong> from Twitter. We only need to connect to these two
  channels with <em>OAuth</em> and select the appropriate Trigger and Action from them respectively. </p>


  <div class='md-10-suffix-1'>
    <figure>
      <img src="/assets/img/post-ifttt-dev/triggers.png" alt="Facebook Triggers"/>
      <figcaption>Figure 1. Facebook Triggers</figcaption>
    </figure>
  </div>



  <div class='md-10-suffix-1'>
    <figure>
      <img src="/assets/img/post-ifttt-dev/actions.png" alt="Twitter Actions"/>
      <figcaption>Figure 2. Twitter Actions</figcaption>
    </figure>
  </div>



<h3> How does it work under the hood? </h3>
Finally, we get to our topic: Development on IFTTT. The way it works is mainly the concern of
channel developers. Their documentation is pretty good, but there are still confusions and undocumented stuff.

Since I've been developing a channel on it for months, and had a lot of email exchanges with their support team regarding
my own confusions and undocumented mechanisms, I think I can help others get a better understanding of how it works.

As we already know, Triggers and Actions are the fundamental building blocks of IFTTT.
We will see how they work in a short while.

<h4> How Trigger works </h4>
So how does IFTTT know that there is a change in your Trigger and that change should fire an Action?
It polls the server of the channel which provided the Trigger every 15 minutes. As a channel developer, we should
return an array of items. Each item should have an unique ID and a timestamp. The IDs are used to prevent Actions from
firing more than once on the same item. So IFTTT keeps a record of all the items it gets, and triggers the Action once there
is a new item (identified by the ID field) coming in.

The items should also contain any data that might be used by the Action. In our Facebook -> Twitter example above, the
Facebook **New status message** Trigger should provide at least the content of the message as a string in addition to
timestamp and ID. The content of the message is called the **Ingredient** which can be used by the Action. In this case,
the Twitter Action **Post a tweet** can use that Ingredient to tweet. A typical data stream you should return looks like
below:



<div class='md-10-suffix-1'>
{% highlight json %}

{
  "data": [
    {
      "content": "It's so great to be home again!",
      "tags": "banksy, brooklyn",
      "posted_at": "2013-12-09T09:23:00-07:00"
      "meta": {
        "id": "14b9-1fd2-acaa-5df5",
        "timestamp": 1383597267
      }
    },
    {
      "content": "School is ending, I'm heading back home soon!",
      "tags": "banksy, nyc",
      "posted_at": "2013-11-29T03:23:00-07:00"
      "meta": {
        "id": "ffb27-a63e-18e0-18ad",
        "timestamp": 1383596355
      }
    }
  ]
}
{% endhighlight %}
</div>



IFTTT has a great test framework that every channel needs to pass before it can be published. One requirements of the
test is that at least 3 items should be returned each poll unless limited by IFTTT itself through a request parameter.
It seems weird at first. You may wonder what if a user hasn't updated his/her Facebook for a long time? What should
we return if we are working for Facebook when IFTTT polls our server? We can return nothing, i.e, an empty list with
'data' as its key. It certainly won't fire any Action, but it's not recommended. We cannot pass the test if we just
return empty data. The recommended way is to return the user's old posts. How many? More than 2
and less than 50, according to the documentation. This policy, according to IFTTT, is to make sure our API behaves
like a timeline of events, not a state engine. Since IFTTT only polls our API every 15 minutes, we need to make sure
that they don't miss anything by returning a list of events.

<h4> Trigger Fields</h4>
There is another term called **Trigger Fields**. It is the user inputs when he/she creates a Trigger. For example,
there's a Facebook Trigger called <strong style="color:#3BB9FF;">New status message by you with hashtag</strong>.
The user needs to specify the name of the hashtag. Only when there is the specified hashtag in the user's status,
will the Trigger fire. When IFTTT polls our API, the Trigger Fields will be passed to us in the POST body, and
we should use it in our server logic to decide what to return.

Trigger Fields can be either static or dynamic. Static ones don't need IFTTT to query our server. In the above example,
the user can just type in a hashtag value himself. For drop down menu, we can provide a list of options in advance
stored in our IFTTT development console. Dynamic ones, on the other hand, require IFTTT to poll our server to get
the values for the user to choose. There is also a protocol specifies how IFTTT polls our server and
how we should respond in their documentation.

<h4>The Real-time API</h4>
When we think about how it works, intuitively we think of posting data to IFTTT's endpoints when a Trigger should fire,
rather than waiting for them to poll our API. Here is when the real-time API comes in. It provides faster response to
user events.

At the time I wrote this post, the real-time API is still very simple and primitive. We only need to return
a list of user IDs whose Triggers have changed.

The user ID comes from the user information endpoint that we provide to IFTTT.
IFTTT polls this endpoint after it acquires the access token through the <em>OAuth2.0</em> process to make sure that
the user's access token is valid. We retrieve the corresponding user with the passed-in access token and return a
user ID and username.

For the real-time API, we just return the same user ID that we return in the user information endpoint. The response
should look like:


<div class='md-10-suffix-1'>
{% highlight json %}
{
  "data": [
    {
      "user_id": "23489759"
    },
    {
      "user_id": "77956024"
    },
    ...
  ]
}
{% endhighlight %}
</div>

Once IFTTT gets our post, it will poll our Trigger API immediately. At the time of writing, we cannot specify
which Trigger to poll. IFTTT will just poll all the Triggers the user has. So the current real-time API behaves more
like a button that we click to force IFTTT to do a Trigger check. It is still our normal Trigger API that does the
actual work. Notice that if IFTTT detects that we are using the real-time API, it will still poll our API regularly
but at a longer period of time.

<h4> How Action works</h4>
Now it should be much easier for us to understand Actions. For each new trigger item, IFTTT will push data to our Action
endpoint with ifttt_source, user and actionFields. We can retrieve the data passed in by Trigger in actionFields. A
sample post looks like:


<div class='md-10-suffix-1'>
{% highlight json %}

POST /ifttt/v1/actions/new_status_update HTTP/1.1
Host: api.example-channel.com
Authorization: Bearer b29a71b4c58c22af116578a6be6402d2
Accept: application/json
Accept-Charset: utf-8
Accept-Encoding: gzip, deflate
Content-Type: application/json
X-Request-ID: 1d21c3cd2ed8441ea269dd554d2c8e54

{
  "actionFields": {
    "title": "New Banksy photo!",
    "body": "Check out a new Bansky photo: http://example.com/images/125"
  },
  "ifttt_source": {
    "id": "2",
    "url": "https://ifttt.com/myrecipes/personal/2"
  },
  "user": {
    "timezone": "Pacific Time (US & Canada)"
  }
}
{% endhighlight %}
</div>


Note that there are also static fields and dynamic options for Actions. For example, if we have an Action
'**Post to channel**' from [Slack], the Slack needs to provide us with the channels we currently have in our
chat room.

<h3> Summary </h3>
Now the way IFTTT works should be pretty simple: It polls our API regularly, if there is new item returned,
it fires an Action. Detailed information can be found in their [documentation].



[Slack]:            https://slack.com/
[documentation]:    https://developers.ifttt.com/docs/api_reference
