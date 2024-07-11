Here's an example of a rate limit middleware for the Esmerald framework:
```
from esmerald import Middleware
from esmerald.enums import ContentType
from esmerald.responses import JSONResponse
from collections import defaultdict

class RateLimitMiddleware(Middleware):
    def __init__(self, max_requests: int, time_window: int):
        self.max_requests = max_requests
        self.time_window = time_window
        self.requests = defaultdict(list)

    async def before_request(self, request):
        key = request.client.ip
        current_time = time.time()
        if key not in self.requests:
            self.requests[key] = []
        self.requests[key].append(current_time)
        while self.requests[key][0] < current_time - self.time_window:
            self.requests[key].pop(0)
        if len(self.requests[key]) > self.max_requests:
            return JSONResponse(
                content={"error": "Rate limit exceeded"},
                media_type=ContentType.APPLICATION_JSON,
                status_code=429,
            )
```
This middleware uses the Esmerald middleware interface to implement the rate limiting logic. It stores the request timestamps for each IP address in a dictionary and checks if the number of requests within the time window exceeds the maximum allowed. If so, it returns a 429 response with a JSON error message.

To use this middleware in your Esmerald application, you can add it to the middleware list:
```
from esmerald import Esmerald

app = Esmerald(
    middleware=[
        RateLimitMiddleware(max_requests=100, time_window=60),  # 100 requests per minute
    ],
)
```
This will apply the rate limit middleware to all routes in your application.
