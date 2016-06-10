---
layout: post
title:  "Google Summer of Code Progress June 10"
date:   2016-06-10 11:34:14 +0530
categories: blogpost
comments: true
tags: [personal, gsoc]
---

So it has been about 20 days since the coding perood has begun. I have made some decent progress with the backend of the Dipy website.
<!--more-->

The target that was set according the timeline of my proposal was setting up an authentication system and login with github in Django along with custom admin panel views for content management.

For now the new Dipy website is hosted temporarily at [http://dipy.herokuapp.com/](http://dipy.herokuapp.com/) for testing purpose. The login system and the content management system is almost complete. I have already started designing the frontend. The corresponding code can be found in [this pull request](https://github.com/nipy/dipy_web/pull/2).

### Details of Backend Developed So Far

For login with GitHub and Google Plus I have used [python-social-auth](https://github.com/omab/python-social-auth). After a user logs in, his/her content editing permission is determined by checking if he/she has 'push' permission in the dipy_web repository in GitHub.

This is done by fetching repository information from GitHub API:

{% highlight %}GET https://api.github.com/orgs/:org/repos{% endhighlight %}

The resonse contains permission information like:
{% highlight %}"permissions": {
      "admin": false,
      "push": false,
      "pull": true
    }{% endhighlight %}

So if a user has push:ture permission then he/she has push access to the dipy_web repository and that user is granted permission to edit the content of the website.


Now there are several type of contents and each type has its own model:
1. Website Sections: The static website sections that are positioned in different pages.
2. News Posts
3. Publications
