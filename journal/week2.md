# Week 2 — Distributed Tracing
## HoneyCombing
We started this week by integrating Honeycomb into our application. Honeycomb is an opentelemetry standard service provided by Honeycomb that'll allow us to maintain observability of our application's main distributed services. With observability we can improve our code by tracing issues that may occur while we allow it to run. 
In order to integrate Honeycomb into our app, we will need to take a few steps to get it all set up. 

### App.py set up 
**the First step** is to add a few dependencies into our `requirements.txt`. These dependencies are OpenTelemetry (informally called OTEL). This an observability framework – software and tools that assist in generating and capturing telemetry data from cloud-native software and is what honeycomb uses to aggregate data from your app. 
```
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```
Once you've added these requirements to the `.txt` file, we can run the usual `pip install -r requirements.txt` within the `/backend-flask/` sub directory. 
![Untitled8](https://user-images.githubusercontent.com/114888726/221869704-266f0155-8520-41a5-aa1f-805924a79914.png)
**NOTE** You must always added the requirements first then run the `pip install`, it is essential you follow these steps sequentially otherwise you are going to have problems with honeycomb, and every other service we are integrating into our application. 

Next, we will import a few libs from Opentelemetry into our `app.py`. This way we can have the required libraries to integrate into our app. 
```
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
```
We are also going to define the initializer for tracing and exporting to Honeycomb. This will also be done in the same `app.py`.
```
# Initialize tracing and an exporter that can send data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
```
![Untitled3](https://user-images.githubusercontent.com/114888726/221871297-6c6e7786-ec14-4da3-a6cd-f2561661a02c.png)
**NOTE** Its best practice to label each function within our application to ensure it works properly. 
**NOTE** This screenshot was taken during the livestream, there are certain additional steps I will outline in the rest of this documentation in order to further explain what we did and what additional functions we added. 

In the screenshot above, you may note that the lines `trace.set_tracer_provider(provider) tracer = trace.get_tracer(__name__)`, are missing. This is because during our livestream, Jessica our honeycomb expert, advised we included these lines in conjunction with some other lines that will allow us to get the backend-flask portion of our app to log data as *STDOUT*. 
```
# Honeycomb: Show this in the logs within the backend-flask app (STDOUT)
simple_processor = SimpleSpanProcessor(ConsoleSpanExporter())
provider.add_span_processor(simple_processor)

trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)
```
![Untitled6](https://user-images.githubusercontent.com/114888726/221873687-233a2503-01f1-469c-9f67-2ae4dfc664af.png)
Your code should look like this after integrating the new function for *STDOUT*.

Lastly, we will have to integrate an automatic instrumentation tool to our `app = Flask (__name__)`. This is essentially the flask framework that allows our app to work. We will add this line beneath it. If you don't add it underneath this segment, the instrumentation will not work thus it won't communicate with Honeycomb:
```
app = Flask(__name__)

# Honeycomb: Initialize automatic instrumentation with Flask
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```
**NOTE** Do not reinclude the `app = Flask(__name__)` in your code, it should already be defined with your `app.py` file. I merely placed it here to demonstrate where the honeycomb line of code belongs.

### Setting env vars
Now that we've defined our code to function with opentelemetry and Honeycomb by extension, we will start to set some environment variables as we will require these particular variables to hold the value of the api key (provided by Honeycomb when we first set up an account), the endpoint, and the name of our service. First will define these vars in our `docker-compose` file. We will add these lines of code here:
![Untitled](https://user-images.githubusercontent.com/114888726/221876849-1e0f1808-afbe-4d78-9a94-5721f0167fed.png)
```
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
OTEL_SERVICE_NAME: "${HONEYCOMB_SERVICE_NAME}"
```
Then we have to define these variables within our gitpod workspace. We will do this with the `export` command including the `gp env` so we can set that variable into this gitpod workspace permanently:
```
export HONEYCOMB_API_KEY=""
export HONEYCOMB_SERVICE_NAME="Cruddur"
gp env HONEYCOMB_API_KEY=""
gp env HONEYCOMB_SERVICE_NAME="Cruddur"
```
![Untitled2](https://user-images.githubusercontent.com/114888726/221877190-35898a6d-ab00-4b5b-bd53-ab038a94b8ac.png)
**NOTE** You must pull your api key from Honeycomb so do not leave the `HONEYCOMB_API_KEY=""` blank, as it must be assigned the API key.

### Creating a custom span
Jessica was kind enough to show us how to create a custom span within our `home_activities.py` file. First will have to import opentelemetry again and then redefine our function to include the functions provided by the Opentelemetry library: 
![Untitled7](https://user-images.githubusercontent.com/114888726/221878447-ca6e672f-de24-45f7-ba83-a1674d43a570.png)
**NOTE** This screenshot is meant to show you where the code positionally belongs. You **must** include the code provided below this picture.
```
from opentelemetry import trace

tracer = trace.get_tracer("Home_activity")

class HomeActivities:
  def run(logger):
    logger.info("HomeActivities")
    with tracer.start_as_current_span("Home_Activities"):
      span = trace.get_current_span()
      now = datetime.now(timezone.utc).astimezone()
      span.set_attribute("app.now", now.isoformat())
```
With this final step, we can now trace our span within Honeycomb!
![Untitled6](https://user-images.githubusercontent.com/114888726/221879100-41c5a9bd-9fa6-4a4e-a4a7-f73eb7c8a1d3.png)

## X-Ray-ing
