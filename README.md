Here's an example of how you can create a rate limit middleware in Esmerald:
```
from esmerald import Middleware
from esmerald.requests import Request
from esmerald.responses import Response
from starlette.datastructures import Headers
from typing import Callable
from functools import wraps
from collections import defaultdict

class RateLimitMiddleware(Middleware):
    def __init__(self, max_calls: int, time_window: int):
        self.max_calls = max_calls
        self.time_window = time_window
        self.cache = defaultdict(int)

    async def __call__(self, request: Request, next: Callable[[Request], Response]) -> Response:
        ip = request.client.host
        current_time = int(request.timestamp)
        if current_time - self.cache[ip] >= self.time_window:
            self.cache[ip] = current_time
            self.cache[(ip, 'count')] = 1
        elif self.cache[(ip, 'count')] < self.max_calls:
            self.cache[(ip, 'count')] += 1
        else:
            headers = Headers({"Retry-After": str(self.time_window)})
            return Response(429, headers=headers)

        return await next(request)

def rate_limit(max_calls: int, time_window: int):
    def decorator(func):
        @wraps(func)
        async def wrapped(*args, **kwargs):
            return await func(*args, **kwargs)
        return wrapped
    return decorator
```
This middleware uses a simple in-memory cache to store the IP addresses and their corresponding request counts. You can customize the `max_calls` and `time_window` parameters to suit your needs.

To use this middleware in your Esmerald API, you can add it to your application like this:
```
from esmerald import Esmerald
from my_rate_limit_middleware import RateLimitMiddleware

app = Esmerald(
    ...
    middleware=[RateLimitMiddleware(max_calls=10, time_window=60)],
    ...
)
```
This will apply the rate limit middleware to all routes in your application. If you want to apply it to specific routes only, you can use the `rate_limit` decorator:
```
from esmerald import Route
from my_rate_limit_middleware import rate_limit

@rate_limit(max_calls=5, time_window=30)
@app.route("/")
async def my_route():
    ...
```
This will apply the rate limit to the specific route only.
