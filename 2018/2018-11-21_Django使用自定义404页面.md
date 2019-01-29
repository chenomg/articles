#### 1. 首先在project/app/views.py中建立404试图  
```python
def page_not_found(request):
    return HttpResponse('Page Not Found!')
```

#### 2. 在project/project/urls.py中指明404使用的视图  
```python
from django.conf.urls import handler404
from app import views
handler404 = views.page_not_found
```

#### 3. 自定404需要在settings.DEBUG = False时生效  
`settings.DEBUG = False`