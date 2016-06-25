---
layout: post
title:  "Google Summer of Code Progress June 24"
date:   2016-06-24 22:40:14 +0530
categories: blogpost
comments: true
tags: [personal, gsoc]
---

This is midterm period and the dipy website has a proper frontend now! And more improvements coming.
![dipy home page screenshot]({{ site.url }}{{ site.baseurl }}/assets/dipy1.png)
<!--more-->

### Progress so far
The custom content management system is improved and has a better frontend now.

Social network feeds for twitter and google plus are now added and a hexagonal gallery is placed in the home page.

The documentation generation script is also updated to upload documentation of different versions separately in github. And the django site is updated by checking the contents of the github repository through github API.

The current pull requests are [#9](https://github.com/nipy/dipy_web/pull/9) and [#1082](https://github.com/nipy/dipy/pull/1082)

You can visit the site under development at [http://dipy.herokuapp.com/](http://dipy.herokuapp.com/)

### Details of content management system
The custom CMS now allows maximum flexibility. It is possible to edit almost every content in the website. Fixed sections, documentation versions, pages, publications, gallery images, news feeds, carousal images, everything can be edited from the admin panel.

![dipy admin panel dashboard screenshot]({{ site.url }}{{ site.baseurl }}/assets/dipy2.png)

One of the most important additions in the CMS is that now we can create any number of pages and we will get a url for that page. So this allows us to create any custom page and link it from anywhere. Also with a single click that page can be put in the nav bar.

![dipy page editor screenshot]({{ site.url }}{{ site.baseurl }}/assets/dipy6.png)


To make the the nav bar dynamic I had to pass the same context to every template. Thankfully this can be achieved with "context_processors" in a dry way.
Also this allows the documentation links to be changed without changing the template.

{% highlight python %}
def nav_pages_processor(request):
    pages = WebsiteSection.objects.filter(section_type="page",
                                          show_in_nav=True)
    all_doc_displayed = DocumentationLink.objects.filter(displayed=True)
    return {'pages_in_nav': pages, 'all_doc_displayed': all_doc_displayed}
{% endhighlight %}

For now the documentations are hosted in the dipy_web github repository. Different versions of documentations are linked automatically into django by checking the content of the repository using github API.

{% highlight python %}
def update_documentations():
    """
    Check list of documentations from gh-pages branches of the dipy_web
    repository and update the database (DocumentationLink model).

    To change the url of the repository in which the documentations will be
    hosted change the DOCUMENTATION_REPO_OWNER and DOCUMENTATION_REPO_NAME
    in settings.py
    """
    url = "https://api.github.com/repos/%s/%s/contents/?ref=gh-pages" % (
        settings.DOCUMENTATION_REPO_OWNER, settings.DOCUMENTATION_REPO_NAME)
    base_url = "http://%s.github.io/%s/" % (
        settings.DOCUMENTATION_REPO_OWNER, settings.DOCUMENTATION_REPO_NAME)
    response = requests.get(url)
    response_json = response.json()
    for content in response_json:
        if content["type"] == "dir":
            version_name = content["name"]
            page_url = base_url + version_name
            try:
                DocumentationLink.objects.get(version=version_name)
            except ObjectDoesNotExist:
                d = DocumentationLink(version=version_name,
                                      url=page_url)
                d.save()
{% endhighlight %}

There is also option of excluding some documentation versions from the admin panel.


### Details of the frontend

Although most parts of the website now have a basic styling, the frontend is still under constant improvement. The progress can be best visualized through some screenshots:

![dipy home page screenshot]({{ site.url }}{{ site.baseurl }}/assets/dipy4.jpeg)

![dipy cite page screenshot]({{ site.url }}{{ site.baseurl }}/assets/dipy3.jpeg)

![dipy support page screenshot]({{ site.url }}{{ site.baseurl }}/assets/dipy5.jpeg)


### What's next

We have to automate the documentation generation process. A build server will be triggered whenever there is a new commit the documentation will be automatically updated in the website. Also there are some command line tools for which the docs must be generated.

I have to include facebook feed in the home page. Also the honeycomb gallery is cool but a carousal with up-to-date contents like upcoming events and news will be more useful. The styling of all parts of the website can be improved.

Then I have to clean up the code a bit and add some more documentation and start testing the internals. After that we have to think of deployment and things like search engine optimization and caching etc.

Will be back with more updates soon! :)