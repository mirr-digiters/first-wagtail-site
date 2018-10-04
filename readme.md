## First Wagtail Site
This is the process I followed that made me familiar with Wagtail CMS.


### Prerequisites
During the creation of the first wagtail site, the following are already installed in my machine:

- Python 3.6.3
- Django 2.0.2
- pip

### Wagtail Installation and First Wagtail Site
The following are the steps I follow for the installation of Wagtail in my machine.

1. I installed Wagtail using pip with the command

    `pip install wagtail`

2. I created the first wagtail site with the command

    `wagtail start firstwagtailsite`

3. (Optional) To check if I have the correct Django version installed, I used the command below. This also checks project dependencies

    `pip install -r requirements.txt`

4. I created the database (SQLite) for the sample site using the command

    `python manage.py migrate`

5. I created an admin user for the site using the command below. This will prompt for some user inputs.

    `python manage.py createsuperuser`

6. I used the command below to run the server

    `python manage.py migrate`
    
7. I tested if the installation is successful by visiting the welcome page http://127.0.0.1:8000. The admin page is at http://127.0.0.1:8000/admin.

### Basic Blog App on the First Wagtail Site

The following are the steps I followed to create a basic blog app on the first wagtail site.

1. Add a body field to the model

    Edit `home/models.py`:
    ```python
    from django.db import models
    from wagtail.core.models import Page
    from wagtail.core.fields import RichTextField
    from wagtail.admin.edit_handlers import FieldPanel


    class HomePage(Page):
        body = RichTextField(blank=True)

        content_panels = Page.content_panels + [
            FieldPanel('body', classname="full"),
        ]
    ```

2. Everytime I have an update on models, I run the following command to update database with the changes

    `python manage.py makemigration`

    `python manage.py migrate`

3. Edit the hompage in the admin page

    `Pages -> Home -> EDIT`

    Add `Hello World` on the BODY section

    
    Figure 1


4. Edit `home/templates/home/home_page.html`:
    ```html
    {% extends "base.html" %}

    {% load wagtailcore_tags %}

    {% block body_class %}template-homepage{% endblock %}

    {% block content %}
        {{ page.body|richtext }}
    {% endblock %}
    ```

5. Check if the changes are reflected in http://127.0.0.1:8000.

6. Create new app named blog using the command

    `python manage.py startapp blog`

7. Add this new app `blog` to `INSTALLED_APPS` in settings `firstwagtailsite/settings/base.py`

8. Create an index page of the blog

    Edit `blog/models.py`

    ```python
    from wagtail.core.models import Page
    from wagtail.core.fields import RichTextField
    from wagtail.admin.edit_handlers import FieldPanel

    class BlogIndexPage(Page):
        intro = RichTextField(blank=True)

        content_panels = Page.content_panels + [
            FieldPanel('intro', classname="full")
        ]
    ```

    `python manage.py makemigration`

    `python manage.py migrate`

9. Create a template `blog/templates/blog/blog_index_page.html`
    ```html
    {% extends "base.html" %}

    {% load wagtailcore_tags %}

    {% block body_class %}template-blogindexpage{% endblock %}

    {% block content %}
        <h1>{{ page.title }}</h1>

        <div class="intro">{{ page.intro|richtext }}</div>

        {% for post in page.get_children %}
            <h2><a href="{% pageurl post %}">{{ post.title }}</a></h2>
            {{ post.specific.intro }}
            {{ post.specific.body|richtext }}
        {% endfor %}

    {% endblock %}
    ```

10. Create a `BlogIndexPage` in the admin page

    Figure 2


11. Edit `blog/models.py`
    ```python
    from django.db import models

    from wagtail.core.models import Page
    from wagtail.core.fields import RichTextField
    from wagtail.admin.edit_handlers import FieldPanel
    from wagtail.search import index

    class BlogIndexPage(Page):
        intro = RichTextField(blank=True)

        content_panels = Page.content_panels + [
            FieldPanel('intro', classname="full")
        ]

    class BlogPage(Page):
        date = models.DateField("Post date")
        intro = models.CharField(max_length=250)
        body = RichTextField(blank=True)

        search_fields = Page.search_fields + [
            index.SearchField('intro'),
            index.SearchField('body'),
        ]

        content_panels = Page.content_panels + [
            FieldPanel('date'),
            FieldPanel('intro'),
            FieldPanel('body', classname="full"),
        ]
    ```

    `python manage.py makemigration`

    `python manage.py migrate`

12. Create a template `blog/templates/blog/blog_page.html`
    ```hmtl
    {% extends "base.html" %}

    {% load wagtailcore_tags %}

    {% block body_class %}template-blogpage{% endblock %}

    {% block content %}
        <h1>{{ page.title }}</h1>
        <p class="meta">{{ page.date }}</p>

        <div class="intro">{{ page.intro }}</div>

        {{ page.body|richtext }}

        <p><a href="{{ page.get_parent.url }}">Return to blog</a></p>

    {% endblock %}
    ```
13. Create blog posts as children of `BlogIndexPage`


    Figure 3

14. Check if the changes are reflected in the blog http://localhost:8000/blog/.

### Concept of Parents and Children

I have applied the concept of parents and children by applying the following changes to our blog app. In our page setup, BlogIndexPage is a parent of BlogPage. The parent uses the `specific` tag to access the fields of its children.
We can also do this using the `with` tag.

Edit `blog_index_page.html` so that it uses the `with` tag

    ```html
    {% extends "base.html" %}

    {% load wagtailcore_tags %}

    {% block body_class %}template-blogindexpage{% endblock %}

    {% block content %}
        <h1>{{ page.title }}</h1>

        <div class="intro">{{ page.intro|richtext }}</div>

        {% for post in page.get_children %}
            {% with post=post.specific %}
                <h2><a href="{% pageurl post %}">{{ post.title }}</a></h2>
                <p>{{ post.intro }}</p>
                {{ post.body|richtext }}
            {% endwith %}
        {% endfor %}

    {% endblock %}
    ```

### Concept of Context Overriding

Overriding context enables modification of QuerySets. This way, we have more control on how data are passed.

1. Edit the `BlogIndexPage` class so that it uses `get_context()`

```python
class BlogIndexPage(Page):
    intro = RichTextField(blank=True)

    def get_context(self, request):
        context = super().get_context(request)
        blogpages = self.get_children().live().order_by('-first_published_at')
        context['blogpages'] = blogpages
        return context
```

2. Slight edit on `blog_index_page.html`:

    Change the line `{% for post in page.get_children %}` to `{% for post in blogpages %}`

### Images

These steps enables the ability to add images to a blog post using functions in Wagtail.

1. Edit `blog/models.py`. Add the import statements, update the `BlogPage` class and add a new class definition  `BlogPageGalleryImage`.

    ```python

    from modelcluster.fields import ParentalKey
    from wagtail.core.models import Page, Orderable
    from wagtail.admin.edit_handlers import FieldPanel, InlinePanel
    from wagtail.images.edit_handlers import ImageChooserPanel

    class BlogPage(Page):
        date = models.DateField("Post date")
        intro = models.CharField(max_length=250)
        body = RichTextField(blank=True)

        search_fields = Page.search_fields + [
            index.SearchField('intro'),
            index.SearchField('body'),
        ]

        content_panels = Page.content_panels + [
            FieldPanel('date'),
            FieldPanel('intro'),
            FieldPanel('body', classname="full"),
            InlinePanel('gallery_images', label="Gallery images"),
        ]

    class BlogPageGalleryImage(Orderable):
        page = ParentalKey(BlogPage, on_delete=models.CASCADE, related_name='gallery_images')
        image = models.ForeignKey(
            'wagtailimages.Image', on_delete=models.CASCADE, related_name='+'
        )
        caption = models.CharField(blank=True, max_length=250)

        panels = [
            ImageChooserPanel('image'),
            FieldPanel('caption'),
        ]
    ```

    `python manage.py makemigration`

    `python manage.py migrate`

2. Add/Edit some blog posts so that it include sample images

Figure 4

3. Edit the `blog_page.html` template so that it will display the uploaded images
    ```html
    {% extends "base.html" %}

    {% load wagtailcore_tags wagtailimages_tags %}

    {% block body_class %}template-blogpage{% endblock %}

    {% block content %}
        <h1>{{ page.title }}</h1>
        <p class="meta">{{ page.date }}</p>

        <div class="intro">{{ page.intro }}</div>

        {{ page.body|richtext }}

        {% for item in page.gallery_images.all %}
            <div style="float: left; margin: 10px">
                {% image item.image fill-320x240 %}
                <p>{{ item.caption }}</p>
            </div>
        {% endfor %}

        <p><a href="{{ page.get_parent.url }}">Return to blog</a></p>

    {% endblock %}
    ```

4. Edit the `blog_page_index.html` template so that it displays thumnails of the images.

    ```html
    {% extends "base.html" %}

    {% load wagtailcore_tags wagtailimages_tags %}

    {% block body_class %}template-blogindexpage{% endblock %}

    {% block content %}
        <h1>{{ page.title }}</h1>

        <div class="intro">{{ page.intro|richtext }}</div>

        {% for post in blogpages %}
            {% with post=post.specific %}
                <h2><a href="{% pageurl post %}">{{ post.title }}</a></h2>

                {% with post.main_image as main_image %}
                    {% if main_image %}{% image main_image fill-160x100 %}{% endif %}
                {% endwith %}

                <p>{{ post.intro }}</p>
                {{ post.body|richtext }}
            {% endwith %}
        {% endfor %}

    {% endblock %}
    ```

### Tags

These steps enables the ability to tag blog posts.

1. Edit `blog/models.py`. Add the import statements, update the `BlogPage` class and add a new class definition  `BlogPageTag`.

    ```python
    from modelcluster.contrib.taggit import ClusterTaggableManager
    from taggit.models import TaggedItemBase
    from wagtail.admin.edit_handlers import FieldPanel, InlinePanel, MultiFieldPanel

    class BlogPageTag(TaggedItemBase):
        content_object = ParentalKey(
            'BlogPage',
            related_name='tagged_items',
            on_delete=models.CASCADE
        )

    class BlogPage(Page):
        date = models.DateField("Post date")
        intro = models.CharField(max_length=250)
        body = RichTextField(blank=True)
        tags = ClusterTaggableManager(through=BlogPageTag, blank=True)

        search_fields = Page.search_fields + [
            index.SearchField('intro'),
            index.SearchField('body'),
        ]

        content_panels = Page.content_panels + [
            MultiFieldPanel([
                FieldPanel('date'),
                FieldPanel('tags'),
            ], heading="Blog information"),
            FieldPanel('intro'),
            FieldPanel('body'),
            InlinePanel('gallery_images', label="Gallery images"),
        ]
    ```

    `python manage.py makemigration`

    `python manage.py migrate`

2. Add/Edit some blog posts so that they include sample tags

    Figure 5

3. Edit the `blog_page.html` template so that it displays the tags. Add the code below.

    ```html
    {% if page.tags.all.count %}
        <div class="tags">
            <h3>Tags</h3>
            {% for tag in page.tags.all %}
                <a href="{% slugurl 'tags' %}?tag={{ tag }}"><button type="button">{{ tag }}</button></a>
            {% endfor %}
        </div>
    {% endif %}
    ```

4. Add a new class definition in `blog/models.py`

    ```python
    class BlogTagIndexPage(Page):

        def get_context(self, request):

            tag = request.GET.get('tag')
            blogpages = BlogPage.objects.filter(tags__name=tag)

            context = super().get_context(request)
            context['blogpages'] = blogpages
            return context
    ```
    
    `python manage.py makemigration`

    `python manage.py migrate`


5. Create a new `BlogTagIndexPage` in the admin page

    Figure 6

6. Create template `blog/blog_tag_index_page.html`

    ```html
    {% extends "base.html" %}
    {% load wagtailcore_tags %}

    {% block content %}

        {% if request.GET.tag|length %}
            <h4>Showing pages tagged "{{ request.GET.tag }}"</h4>
        {% endif %}

        {% for blogpage in blogpages %}

            <p>
                <strong><a href="{% pageurl blogpage %}">{{ blogpage.title }}</a></strong><br />
                <small>Revised: {{ blogpage.latest_revision_created_at }}</small><br />
                {% if blogpage.author %}
                    <p>By {{ blogpage.author.profile }}</p>
                {% endif %}
            </p>

        {% empty %}
            No pages found with that tag.
        {% endfor %}

    {% endblock %}
    ```

7. Test if changes are reflected. Visit a blog post and click on one of the assigned tags

### Wagtail Snippets and Categories

The concept of snippets can be applied on the sample blog app by following the steps below. Blog categories are used to demonstrate this concept.

1. Define a standard Django model `BlogCategory`.

    ```python
    from wagtail.snippets.models import register_snippet

    @register_snippet
    class BlogCategory(models.Model):
        name = models.CharField(max_length=255)
        icon = models.ForeignKey(
            'wagtailimages.Image', null=True, blank=True,
            on_delete=models.SET_NULL, related_name='+'
        )

        panels = [
            FieldPanel('name'),
            ImageChooserPanel('icon'),
        ]

        def __str__(self):
            return self.name

        class Meta:
            verbose_name_plural = 'blog categories'
    ```
    
    `python manage.py makemigration`

    `python manage.py migrate`

2. A new option named `Snippets` is enabled in the admin menu. Add some categories using this option.

3. Edit `blog/models.py`. Add new import statements and edit `BlogPage`

    ```python
    from django import forms

    from modelcluster.fields import ParentalKey, ParentalManyToManyField

    class BlogPage(Page):
        date = models.DateField("Post date")
        intro = models.CharField(max_length=250)
        body = RichTextField(blank=True)
        tags = ClusterTaggableManager(through=BlogPageTag, blank=True)
        categories = ParentalManyToManyField('blog.BlogCategory', blank=True)

        search_fields = Page.search_fields + [
            index.SearchField('intro'),
            index.SearchField('body'),
        ]

        content_panels = Page.content_panels + [
            MultiFieldPanel([
                FieldPanel('date'),
                FieldPanel('tags'),
                FieldPanel('categories', widget=forms.CheckboxSelectMultiple),
            ], heading="Blog information"),
            FieldPanel('intro'),
            FieldPanel('body'),
            InlinePanel('gallery_images', label="Gallery images"),
        ]

    ```
    `python manage.py makemigration`

    `python manage.py migrate`

4. Edit `blog_page.html` template so that it will display the categories. Insert the code below.

    ```html
    {% with categories=page.categories.all %}
        {% if categories %}
            <h3>Posted in:</h3>
            <ul>
                {% for category in categories %}
                    <li style="display: inline">
                        {% image category.icon fill-32x32 style="vertical-align: middle" %}
                        {{ category.name }}
                    </li>
                {% endfor %}
            </ul>
        {% endif %}
    {% endwith %}
    ```

5. Add/Edit some blog posts so that they are assigned in some categories

6. Test if changes are reflected. Visit a blog post and check if there are categories displayed.

### Deploying the First Wagtail Site to Heroku

The method used for deployment is through `Heroku CLI`.

1. Install `Heroku CLI` (https://devcenter.heroku.com/articles/heroku-cli).
2. Log in using your Heroku account using the command below

    `heroku login`

3. Make sure that `gunicorn` is installed on the Heroku instance. Add `gunicorn` to `requirements.txt` to ensure this.

    `requirements.txt`:
    ```
    Django>=2.0,<2.1
    wagtail>=2.2,<2.3
    gunicorn>=19.9.0
    ```

4. Create a `Procfile` on the root directory and add the command below

    `Procfile`:
    ```
    web: gunicorn firstwagtailsite.wsgi --log-file -
    ```

5. Go to your `first wagtail site` folder. If not yet in a repository, initialize git and point the repository to Heroku instance. Add and commit the changes, then push them to heroku master.

    `cd first/wagtail/site/folder`

    `git init`

    `heroku git:remote -a first-wagtail-site`

    `git add -A`

    `git commit -am "initial commit"`

    `git push heroku master`

6. Test if the deployment is successful by visiting the Heroku app 

    `heroku open`

### Static Assets on Heroku Using Whitenoise

1. Make sure that `whitenoise` is installed on the Heroku instance. Add `whitenoise` to `requirements.txt` to ensure this.

    `requirements.txt`:
    ```
    Django>=2.0,<2.1
    wagtail>=2.2,<2.3
    gunicorn>=19.9.0
    whitenoise>=4.1
    ```

2. Add the following to `dev.py` settings file.
    ```python
    STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'

    COMPRESS_OFFLINE = True
    COMPRESS_CSS_FILTERS = [
        'compressor.filters.css_default.CssAbsoluteFilter',
        'compressor.filters.cssmin.CSSMinFilter',
    ]
    ```
### References
- http://docs.wagtail.io/en/latest/getting_started/tutorial.html
- https://devcenter.heroku.com/articles/python-gunicorn