# Error Handling in Django Views (Simple & Deep Understanding)

In Django, error handling in views is essential to ensure a smooth user experience and proper debugging. Let's break it down step by step.

## 1. Understanding How Errors Occur in Django Views
A Django view is a function or class-based method that processes a request and returns a response. Errors occur when:
- A user provides invalid input.
- A database query fails.
- An object is not found (e.g., accessing a non-existent ID).
- The server encounters an internal error.

---

## 2. Handling Errors Using `try-except`
The simplest way to handle errors is by using `try-except` blocks.

### Example: Handling Database Query Errors
```python
from django.http import JsonResponse
from myapp.models import Student

def student_detail(request, student_id):
    try:
        student = Student.objects.get(id=student_id)
        return JsonResponse({'name': student.name, 'age': student.age})
    except Student.DoesNotExist:
        return JsonResponse({'error': 'Student not found'}, status=404)
    except Exception as e:
        return JsonResponse({'error': str(e)}, status=500)
```
✅ **Explanation**:
- We attempt to fetch a student by ID.
- If the student is not found, we return a `404` response.
- If any other unexpected error occurs, we return a `500` response.

---

## 3. Using Django’s Built-in Exception Handling
Django provides built-in exceptions that we can use directly.

### Common Django Exceptions
| Exception | Description |
|-----------|------------|
| `Http404` | Raised when a requested object is not found. |
| `PermissionDenied` | Raised when the user does not have permission. |
| `SuspiciousOperation` | Raised for suspicious user actions (e.g., CSRF failure). |
| `BadRequest` | Raised when the request is malformed. |

### Example: Raising `Http404` Manually
Instead of handling `DoesNotExist`, we can use `get_object_or_404`:
```python
from django.shortcuts import get_object_or_404
from django.http import JsonResponse
from myapp.models import Student

def student_detail(request, student_id):
    student = get_object_or_404(Student, id=student_id)
    return JsonResponse({'name': student.name, 'age': student.age})
```
✅ **Why use `get_object_or_404`?**
- It automatically raises `Http404` if the object is not found.
- It simplifies code readability.

---

## 4. Custom Error Pages (404, 500, 403, etc.)
Django allows you to create custom error pages for common errors.

### Creating a Custom 404 Page
#### 1. **Create `templates/404.html`**
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
#### 2. **Modify `urls.py`**
```python
handler404 = 'myapp.views.custom_404'
```
#### 3. **Define the Custom View (`views.py`)**
```python
from django.shortcuts import render

def custom_404(request, exception):
    return render(request, '404.html', status=404)
```
✅ **How it Works?**
- When a 404 error occurs, Django automatically calls `custom_404`.
- It returns the `404.html` template with a 404 status.

Similarly, you can define custom views for:
```python
handler500 = 'myapp.views.custom_500'  # Internal Server Error
handler403 = 'myapp.views.custom_403'  # Permission Denied
handler400 = 'myapp.views.custom_400'  # Bad Request
```

---

## 5. Using Django Middleware for Global Error Handling
If you want a centralized way to handle errors, you can use middleware.

### Creating Custom Middleware for Error Logging
#### 1. **Create a Middleware File (`middleware.py`)**
```python
import logging
from django.http import JsonResponse

logger = logging.getLogger(__name__)

class CustomExceptionMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        try:
            response = self.get_response(request)
            return response
        except Exception as e:
            logger.error(f"Error: {str(e)}")
            return JsonResponse({'error': 'Something went wrong'}, status=500)
```
#### 2. **Register it in `settings.py`**
```python
MIDDLEWARE = [
    'myapp.middleware.CustomExceptionMiddleware',
    # Other middleware...
]
```
✅ **Why use Middleware?**
- Centralized error handling.
- Logs all exceptions in one place.
- Avoids repeating `try-except` in multiple views.

---

## 6. Logging Errors for Debugging
Instead of just returning an error, it's good to log them for debugging.

### Configuring Logging in `settings.py`
```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'ERROR',
            'class': 'logging.FileHandler',
            'filename': 'errors.log',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file'],
            'level': 'ERROR',
            'propagate': True,
        },
    },
}
```
✅ **What this does?**
- Logs all errors into `errors.log`.
- Helps in debugging production issues.

---

## 7. Debug Mode vs. Production Mode
Django behaves differently in development (`DEBUG=True`) vs. production (`DEBUG=False`).

### **When `DEBUG=True`**
- Django shows detailed error pages with stack traces.
- Useful for debugging during development.

### **When `DEBUG=False`**
- Django hides detailed errors.
- Custom error pages are used.
- Errors should be logged for admins.

✅ **Pro Tip**: Always set `DEBUG=False` in production and configure logging properly.

---

## Conclusion
### ✅ **Best Practices for Error Handling in Django**
1. Use `try-except` for critical operations (like DB queries).
2. Use Django’s built-in `get_object_or_404` instead of handling `DoesNotExist`.
3. Create custom error pages for a better user experience.
4. Implement middleware for centralized error handling.
5. Log errors properly using Django’s logging system.
6. Always set `DEBUG=False` in production to prevent sensitive data exposure.

---
