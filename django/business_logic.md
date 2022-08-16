# Business logic in Django

This is a guide of conventions for storing business logic in [Django](https://www.djangoproject.com/) projects at [SoftFormance](https://softformance.com).

Contents:

- [Services vs QuerySets/Managers](#services-vs-querysetsmanagers)
- [Useful links](#useful-links)


## Services vs QuerySets/Managers

> Weekend hot take: “hide data access behind a service layer” is the “write getter/setter methods wrapping private attributes” of web application architecture.
> 
> Which is to say, both can solve niche problems you might not have, but are wrongly promoted as universal best practices.

(c) [James Bennett](https://twitter.com/ubernostrum/status/1239009718519001089)

James Bennett has a good post ["Against service layers in Django"](https://www.b-list.org/weblog/2020/mar/16/no-service/).
Also, there is a followup post ["More on service layers in Django"](https://www.b-list.org/weblog/2020/mar/23/still-no-service/).

Comparing where to put business logic in Django you can find [that article](https://sunscrapers.com/blog/where-to-put-business-logic-django/)



## Models/QuerySets/Managers

Usually, we add methods to our models to encapsulate business logic and hide implementation details.
Using low-level ORM code directly in a view is an anti-pattern.

- Make sure you are aware of [Manager topic](https://docs.djangoproject.com/en/stable/topics/db/managers/). Pay attention to [Default managers](https://docs.djangoproject.com/en/stable/topics/db/managers/#default-managers) and [Base managers](https://docs.djangoproject.com/en/stable/topics/db/managers/#base-managers)
- Use only one default Manager `objects` in your model. Django developers are used to thinking of Model.objects as the "gateway" to the table. Here is a showing a [different approaches how we can use Managers/QuerySets](https://www.dabapps.com/blog/higher-level-query-api-django-orm/)
- Use [Proxy model](https://docs.djangoproject.com/en/4.0/topics/db/models/#proxy-models-1) with additional Manager
- Do not modilfy `get_queryset()` unless you understand what you are doing. If you don't completly understand it, then re-read again [topic](https://docs.djangoproject.com/en/stable/topics/db/managers/)
- Start naming of every new queryset method which add annotation with `with`. Example: `def with_responses(self):`
- Here is a good [Django ORM Cookbook](https://books.agiliq.com/projects/django-orm-cookbook/en/latest/index.html) with answers to common problems
- [Example of design issue](https://www.dabapps.com/blog/django-models-and-encapsulation/) in large Django applications
- There was [mention](https://forum.djangoproject.com/t/structuring-large-complex-django-projects-and-using-a-services-layer-in-django-projects/1487/21) of `use_cases.py` module which can be used as a simple layer between Views and Models/Managers/QuerySets.


## Useful links

- [Structuring large/complex Django projects, and using a services layer in Django projects](https://forum.djangoproject.com/t/structuring-large-complex-django-projects-and-using-a-services-layer-in-django-projects/1487)
- [Building a higher-level query API: the right way to use Django's ORM](https://www.dabapps.com/blog/higher-level-query-api-django-orm/) - comparing different approaches of custom/multiple Managers, custom QuerySet.
- [Django models, encapsulation and data integrity](https://www.dabapps.com/blog/django-models-and-encapsulation/) - great explanation of rule "fat models, thin views".
- [Thin views](https://spookylukey.github.io/django-views-the-right-way/thin-views.html) - what **not** to put in a view.
- [Against service layers in Django](https://www.b-list.org/weblog/2020/mar/16/no-service/) - good explain why you don't need "service layer" in Django project
- [More on service layers in Django](https://www.b-list.org/weblog/2020/mar/23/still-no-service/) - followup of previous post.
- [The Dramatic Benefits of Django Subqueries and Annotations](https://hansonkd.medium.com/the-dramatic-benefits-of-django-subqueries-and-annotations-4195e0dafb16)
- [Solving Performance Problems in the Django ORM](https://hansonkd.medium.com/performance-problems-in-the-django-orm-1f62b3d04785)
- [Django ORM Cookbook](https://books.agiliq.com/projects/django-orm-cookbook/en/latest/index.html)
- [Separation of business logic and data access in Django](https://stackoverflow.com/a/12857584)
- [Django for Startup Founders: A better software architecture for SaaS startups and consumer apps](https://alexkrupp.typepad.com/sensemaking/2021/06/django-for-startup-founders-a-better-software-architecture-for-saas-startups-and-consumer-apps.html)
- [Hacker News: thread about "fat models vs service layers"](https://news.ycombinator.com/item?id=23325113)
- [Reddit: Business logic in django](https://www.reddit.com/r/django/comments/kce30r/business_logic_in_django/)
- [Medium: Business Logic in Django projects](https://jairvercosa.medium.com/business-logic-in-django-projects-7fe700db9b0a) - M**U**VT - Use cases execute the business logic and run the side effects. 
- [Django Forum: Where to put business logic in Django?](https://forum.djangoproject.com/t/where-to-put-business-logic-in-django/282)
