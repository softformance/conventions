# Django apps

This is a guide of conventions of [Django](https://www.djangoproject.com/) apps at [SoftFormance](https://softformance.com).

Contents:

- [Django apps](#django-apps)
- [Recommended apps](#recommended-apps)
- [Useful links](#useful-links)


## Django apps

### Why use Django apps
- Separation of concerns - hold 1 app in your head at a time
- Consistensy - every app looks the same
- Automation - example: auto-discovery and plugins

### How to scale Django apps and common app patterns
- When we make a new app? Most new things should be a new app. Easier to combine than to separate later.
- Common app patterns
    - New feature. e.g. Comments on a Blog website
    - Internal vs External. e.g. Comment moderation interface
    - Multi-part features. e.g. Each social sign-in system, Google, Apple, GitHub


Example of nesting apps:
```
-> orders/
   -> orders_royal_mail/
   -> orders_ukmail/
   -> orders_collect_plus/
   -> orders_fraud/
      -> orders_fraud_postcode_flags/
```


## Recommended apps

[DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)! Django has [many apps available](https://djangopackages.org/), and a lot of them can help you to develop your project in a cleaner and faster way. Here is a list of curated list of apps that can help you:

https://raw.githubusercontent.com/sthima/django-styleguide/master/README.md
.....


## Useful links

- [Video: Dan Palmer - Scaling Django to 500 apps](https://www.youtube.com/watch?v=NsHo-kThlqI)
- [DjangoProject Forum: Why do we need apps?](https://forum.djangoproject.com/t/why-do-we-need-apps/827)
