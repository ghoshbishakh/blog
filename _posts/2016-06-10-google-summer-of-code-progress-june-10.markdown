---
layout: post
title:  "Google Summer of Code Progress June 10"
date:   2016-06-10 22:40:14 +0530
categories: blogpost
comments: true
tags: [personal, gsoc]
---

So it has been about 20 days since the coding period has begun. I have made some decent progress with the backend of the Dipy website.
<!--more-->

The target that was set according the timeline of my proposal was setting up an authentication system and login with github in Django along with custom admin panel views for content management.

For now the new Dipy website is hosted temporarily at [http://dipy.herokuapp.com/](http://dipy.herokuapp.com/) for testing purpose. The login system and the content management system is almost complete. I have already started designing the frontend. The corresponding code can be found in [this pull request](https://github.com/nipy/dipy_web/pull/2).

### Details of Backend Developed So Far

For login with GitHub and Google Plus I have used [python-social-auth](https://github.com/omab/python-social-auth). After a user logs in, his/her content editing permission is determined by checking if he/she has 'push' permission in the dipy_web repository in GitHub.

This is done by fetching repository information from GitHub API with the user's access token:

{% highlight bash %}GET https://api.github.com/orgs/:org/repos{% endhighlight %}

The resonse contains permission information like:
{% highlight json %}"permissions": {
      "admin": false,
      "push": false,
      "pull": true
    }{% endhighlight %}

So if a user has push:true permission then he/she has push access to the dipy_web repository and that user is granted permission to edit the content of the website.


Now there are several type of contents and each type has its own model:

1. Website Sections: The static website sections that are positioned in different pages.

2. News Posts

3. Publications


#### Website Sections:

The website sections contains some identifiers like which page it belongs to and in which position it should be placed. The content body of the website section is written in markdown. To change the markdown to HTML the [markdown](https://pythonhosted.org/Markdown/) library is used. The model's save() method is overrided so that each time it is edited, the new HTML is generated from the markdown. The HTML is filtered using the [bleach](http://bleach.readthedocs.io/) library.

{% highlight python %}
def save(self, *args, **kwargs):
    html_content = markdown.markdown(self.body_markdown,
                                     extensions=['codehilite'])
    print(html_content)
    # bleach is used to filter html tags like <script> for security
    self.body_html = bleach.clean(html_content, allowed_html_tags,
                                  allowed_attrs)
    self.modified = datetime.datetime.now()
    # Call the "real" save() method.
    super(WebsiteSection, self).save(*args, **kwargs)
{% endhighlight %}

There is requirement of embedding youtube videos in the markdown content of website sections. But allowing ifames in markdown would allow embedding any kind of content from any arbitrary source. So I wrote a custom template filter for converting the youtube links into embed codes. This also makes it simple to embed the videos as only pasting the url is all that the user needs to do.

{% highlight python %}
register.filter(name='youtube_embed_url')
def youtube_embed_url(value):
    exp = re.compile(r'((http|https)\:\/\/www\.youtube\.com\/watch\?v=([a-zA-Z0-9]*))')
    matches = exp.findall(value)
    processed_str = value
    template = '<div class="youtube-wrapper"><iframe class="youtube-embed" width="640" height="360" \
                src="https://www.youtube.com/embed/%s?rel=0&modestbranding=1" \
                frameborder="0" allowfullscreen></iframe></div>'
    for match in matches:
        processed_str = processed_str.replace(match[0], template % match[2])
    return processed_str
{% endhighlight %}

#### News Posts:

News posts are simple models for storing news with post dates. Most recent news are displayed in the home page.

#### Publications:

Publications are Journal/Conference/Book Chapter publications or any other literature about Dipy that people may want to know about or cite.

One important information about the publications that people often seek is the bibtex. Also the publication information can be extracted from the bibtex only. So instead of entering all details of the publications in a form, the content editor can only enter the bibtex and the model can be automatically populated by parsing the bibtex. For parsing bibtex instead of writing a new parser from scratch I have used [bibtexparser](https://pypi.python.org/pypi/bibtexparser/0.6.2) library.

Another property of the publications is that they can be marked as highlighted and the highlighted publications can be displayed separately in the home page or anywhere else.


### Some more thoughts on the GSOC journey so far

I just love it :) Especially because of my mentors. We have weekly video call meetings where a very specific set of targets are set for the coming week. This helps to maintain my focus in a particular direction. Also there is regular exchange of mails throught the week. They also go through my code every week and give feedback, so I am always trying to be very careful and make less mistakes :P

They inspire me to work harder.