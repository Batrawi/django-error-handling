# Handling Errors in Views

In a Django application, a view function is where all the processing is done. It receives the request and formulates the response.

In this reading, you’ll learn how Django handles runtime errors or exceptions.

The response from the view is an object of `HttpResponse`. Its contents are also associated with a respective status code.

For example, status code `404` implies that the resource requested by the client cannot be found. Django has a generic `HttpResponseNotFound` class. You can return its object to convey the appropriate message.

```python
from django.http import HttpResponse, HttpResponseNotFound 
def my_view(request): 
    # ... 
    if condition==True: 
        return HttpResponseNotFound('<h1>Page not found</h1>') 
    else: 
        return HttpResponse('<h1>Page was found</h1>') 
```

You can also return an `HttpResponse` with the status code denoting the type of error.

```python
from django.http import HttpResponse 
def my_view(request): 
    # ... 
    if condition==True: 
        return HttpResponse('<h1>Page not found</h1>', status=404) 
    else: 
        return HttpResponse('<h1>Page was found</h1>') 
```

The main difference in using `HttpResponseNotFound` as opposed to `HttpResponse` with a status code is that Django internally sends an error code `404`. The appropriate page for `404` can then be configured and rendered, else the browser displays its default `404` view. 

A common cause of the `404` status code is the user entering an incorrect URL. Django makes it easy to work with this error code. Instead of returning `HttpResponseNotFound()`, you can raise `Http404`, which displays a standard error page with the status.

### Handling Database Query Errors
Consider the following scenario, where you have a `Product` model in the app. The user wants the details of a product with a specific `Product ID`. In the following view function, `id` is the parameter obtained from the URL. It tries to determine whether any product with the given `id` is available. If not, the `Http404` exception is raised.

```python
from django.http import Http404, HttpResponse 
from .models import Product 

def detail(request, id): 
    try: 
        p = Product.objects.get(pk=id) 
    except Product.DoesNotExist: 
        raise Http404("Product does not exist") 
    return HttpResponse("Product Found") 
```

Just like `HttpResponseNotFound`, there are a number of other predefined classes such as `HttpResponseBadRequest`, `HttpResponseForbidden`, and so on.

---

## Custom Error Pages
If you want to show your own error page whenever the user encounters a `404` error, you must create a `404.html` page and put it in the `project/templates` folder.

```html
<!DOCTYPE html>
<html>
<head>
    <title>Page Not Found</title>
</head>
<body>
    <h1>Oops! Page not found.</h1>
    <p>The page you are looking for doesn't exist.</p>
</body>
</html>
```

Modify `urls.py`:
```python
handler404 = 'myapp.views.custom_404'
```

Define the custom view (`views.py`):
```python
from django.shortcuts import render

def custom_404(request, exception):
    return render(request, '404.html', status=404)
```

Similarly, you can define custom views for:
```python
handler500 = 'myapp.views.custom_500'  # Internal Server Error
handler403 = 'myapp.views.custom_403'  # Permission Denied
handler400 = 'myapp.views.custom_400'  # Bad Request
```

---

## Displaying Error Messages in the Browser
Usually, the Django development server is in `DEBUG` mode, which shows the error's traceback instead of the exception.

To render the custom exception message, the `DEBUG` variable in the project’s settings should be set to `False`.

```python
# settings.py  
DEBUG = False 
```

---

## Exception Classes
Django’s exception classes are defined in the `django.core.exceptions` module.

### Some important exception types:
- `ObjectDoesNotExist`: All exceptions of `DoesNotExist` are inherited from this base exception.
- `EmptyResultSet`: Raised if a query does not return any result.
- `FieldDoesNotExist`: Raised when the requested field does not exist.
- `MultipleObjectsReturned`: Raised when a query expected to return a single object returns multiple objects.
- `PermissionDenied`: Raised when a user does not have permission to perform the requested action.
- `ViewDoesNotExist`: Raised when a requested view does not exist, possibly due to incorrect mapping in the URLconf.

Example of `FieldDoesNotExist`:
```python
try: 
    field = model._meta.get_field(field_name) 
except FieldDoesNotExist: 
    return HttpResponse("Field Does not exist") 
```

Example of `PermissionDenied`:
```python
def myview(request): 
    if not request.user.has_perm('myapp.view_mymodel'): 
        raise PermissionDenied() 
    return HttpResponse() 
```

When a certain view is called with a `POST` or `PUT` request, the request body is populated by form data. Django’s Form API defines various fields specific to the type of data stored. These fields have built-in validators. The `is_valid()` method returns `True` if the validations pass; otherwise, an exception can be raised.

```python
def myview(request):   
    if request.method == "POST":   
        form = MyForm(request.POST)   
        if form.is_valid():   
            # Process the form data 
        else:   
            return HttpResponse("Form submitted with invalid data") 
```

In addition to Django's built-in exceptions, you can also handle standard Python exceptions and database-related exceptions.

---

## Conclusion
### ✅ Best Practices for Error Handling in Django
1. Use `try-except` for critical operations (like database queries).
2. Use Django’s built-in `get_object_or_404` instead of handling `DoesNotExist` manually.
3. Create custom error pages for a better user experience.
4. Implement middleware for centralized error handling.
5. Log errors properly using Django’s logging system.
6. Always set `DEBUG=False` in production to prevent sensitive data exposure.
