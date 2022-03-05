# Celery

This is a guide of conventions for [celery](https://github.com/celery/celery) at [SoftFormance](https://softformance.com).

Contents:

- [Function definition](#function-definition)
- [Settings](#settings)
- [Error handling](#error-handling)
- [Caveats](#caveats)
- [Useful links](#useful-links)
- [Extensions](#extensions)


## Function definition

- Each task should end with `_task`. Example: `send_email_to_customer_task`, `send_reminder_about_appointment_task`,
  `set_buyer_credit_rating_task`.
- Think about naming of task before deploying it to production. More details at [caveats](#caveats).
- Use `bind=True` keyword argument for task, so you can always access task in `self`. More details at
  [Celery docs](https://docs.celeryproject.org/en/stable/userguide/tasks.html#bound-tasks). Example:
```python
logger = get_task_logger(__name__)

@celery_app.task(bind=True)
def send_email_to_customer_task(self, **kwargs):
    logger.info(self.request.id)
```
- Use tiny and simple data structure as task arguments instead of full objects.
```python
@celery_app.task
def send_reminder_about_appointment_task(appointment_id: int, **kwargs):
    appointment = Appointment.objects.get(id=appointment_id)
    ...
```
- Keep minimum of logic inside task. Think about celery task more as a trigger for running another code.
  Good example can be next code:
```python
@celery_app.task
def send_reminder_about_appointment_task(appointment_id: int, **kwargs):
    appointment = Appointment.objects.get(id=appointment_id)
    appointment.send_reminder_email()
```
- Keep backward compatibility for celery tasks between releases with `**kwargs`

## Settings

- Set global [task_time_limit](https://docs.celeryproject.org/en/stable/userguide/configuration.html#task-time-limit)
  and [task_soft_time_limit](https://docs.celeryproject.org/en/stable/userguide/configuration.html#task-soft-time-limit).
  In case of `SoftTimeLimitExceeded` you can recover if things take longer than expected.
- Use [task queues](https://docs.celeryproject.org/en/latest/userguide/routing.html), Luke.
  It will allow you to scale things in the future.

## Error handling

- Ideally, tasks should be [idempotent](https://docs.celeryproject.org/en/stable/glossary.html#term-idempotent) and [atomic](https://en.wikipedia.org/wiki/Atomicity_(database_systems)).

Example of idempotent task:
```python
@celery_app.task
def add(x, y):
    return x + y
```
As you can see result will be the same not matter how many times you call that task

Example of atomic task:
```python
@celery_app.task
def send_newsletter_task(email: str, **kwargs):
    Newsletter.send_email_to(email)


@celery_app.task
def send_newsletters_task(**kwargs):
    emails = User.objects.subscribed_to_newsletter()
    for email in emails:
        send_newsletter_task.delay(email)
```
- Use timeout for global tasks and for concrete tasks
> Not setting a timeout for a task is evil.
> This means that you do not understand what is happening in the task, how business logic should work.

Taken from [50 shades of celery](https://sudonull.com/post/6810-50-shades-of-celery)
- Follow [Retry policy](https://docs.celeryproject.org/en/stable/userguide/tasks.html#retrying) for task.
  Get familiar with `autoretry_for`, `retry_backoff`, `retry_backoff_max` and `retry_jitter`
- If your tasks depend on another service, like making a request to an API,
  then it's a good idea to use exponential backoff to avoid overwhelming the service with your requests
  (it is known as [Thundering herd problem](https://en.wikipedia.org/wiki/Thundering_herd_problem))

## Caveats

- Backward compatibility
> We had several interesting side effects with a deployment application.
> It doesn't matter what type of deployment you use (blue + green or rolling update),
> there will always be a situation when the old service code creates messages for the new worker code,
> and vice versa, the old worker receives messages from the new service code because
> it rolls out "first" and there the traffic went.

Taken from [50 shades of celery](https://sudonull.com/post/6810-50-shades-of-celery)
- [Avoid launching synchronous subtasks](https://docs.celeryproject.org/en/stable/userguide/tasks.html#avoid-launching-synchronous-subtasks)
- [Task transaction implementation](https://docs.celeryproject.org/en/stable/userguide/tasks.html#database-transactions)
- [Visibility timeout for Redis as a broker](https://docs.celeryproject.org/en/stable/getting-started/backends-and-brokers/redis.html#visibility-timeout)

## Useful links

- [Celery: A Few Gotchas Explained](http://www.ines-panker.com/2020/10/28/celery-explained.html)
- [Why is Celery not running my task?](https://www.lorenzogil.com/blog/2020/03/01/celery-tasks/) - visual explanation how tasks are distributed
- [Celery - Distributed Task Queue](https://docs.celeryproject.org/en/stable/index.html) - official docs
- [50 shades of celery](https://sudonull.com/post/6810-50-shades-of-celery) - excellent advice how to work with celery on own experience of author
- [Celery tasks checklist](https://devchecklists.com/celery-tasks-checklist/)
- [Celery in production: Three more years of fixing bugs](https://medium.com/squad-engineering/celery-in-production-three-more-years-of-fixing-bugs-2ee462cef39f)
- [5 tips for writing production-ready Celery tasks](https://blog.wolt.com/engineering/2021/09/15/5-tips-for-writing-production-ready-celery-tasks/)
- [Automatically Retrying Failed Celery Tasks](https://testdriven.io/blog/retrying-failed-celery-tasks/) - one of "Django + Celery" series from [testdriven.io](https://testdriven.io/)
- [Retry Celery tasks with exponential back off](https://stackoverflow.com/questions/9731435/retry-celery-tasks-with-exponential-back-off)
- [Celery Task Retry Guide by Ines Panker](https://ines-panker.medium.com/celery-task-retry-guide-e47e184a9198)
- [Celery in the wild: tips and tricks to run async tasks in the real world](https://www.vinta.com.br/blog/2018/celery-wild-tips-and-tricks-run-async-tasks-real-world/)
- [Common Issues Using Celery (And Other Task Queues)](https://adamj.eu/tech/2020/02/03/common-celery-issues-on-django-projects/)
- [Celery: Django Admin Action to Manually Retry Tasks](https://medium.com/@sameer_kumar/celery-django-admin-action-to-manually-retry-tasks-90b8013b0b8f)
- [Workers, Pool and Concurrency configurations of Python Celery](https://medium.com/analytics-vidhya/python-celery-explained-for-beginners-to-professionals-part-3-workers-pool-and-concurrency-ef0522e89ac5)
- [How we scaled Celery for our Django app](https://vedantsopinions.medium.com/how-we-scaled-celery-for-our-django-app-da2465a3a6be)


## Extensions

- [celery-singleton](https://github.com/steinitzu/celery-singleton)
- [celery-batches](https://github.com/clokep/celery-batches)
- [single-beat](https://github.com/ybrs/single-beat)
- [flower](https://github.com/mher/flower)
- [django-celery-beat](https://github.com/celery/django-celery-beat)
- [django-celery-results](https://github.com/celery/django-celery-results)
- [django-celery-email](https://github.com/pmclanahan/django-celery-email)
- [redbeat](https://github.com/sibson/redbeat)
- [celery-director](https://github.com/ovh/celery-director)
- [celery-exporter](https://github.com/danihodovic/celery-exporter)
- [celery-dyrygent](https://github.com/ovh/celery-dyrygent)
- [celery-types](https://github.com/sbdchd/celery-types)
