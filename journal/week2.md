# Week 2 — Distributed Tracing
## HoneyCombing
We started this week by integrating Honeycomb into our application. Honeycomb is an opentelemetry standard service provided by Honeycomb that'll allow us to maintain observability of our application's main distributed services. With observability we can improve our code by tracing issues that may occur while we allow it to run. 
In order to integrate Honeycomb into our app, we will need to take a few steps to get it all set up. 

### App.py set up 
______________________
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
___________________________
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
__________________________________
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
AWS Xray is similar to Honeycomb in that it using distributed tracing to your applications in order for you to gain insights on latency and performance of your applications. With AWS Free Tier, the first 100,000 traces are recorded for **FREE**, which is great as you want to leverage the free tier as much as possible during this bootcamp. We will be integrating X-ray into our application to expand our observability to our app. 

### Adding Requirements
____________________________
The first thing to do is to add the `aws-xray-sdk` to our `requirements.txt`. Be sure that you have `cd` into your `/backend-flask/` and as with every other addition to our requirements, we will run our favorite installation tool, `pip install -r requirements.txt` within that `backend-flask` directory. 
![Untitled1](https://user-images.githubusercontent.com/114888726/221943058-abad829c-e4d3-415a-8366-91d80d6d9ee5.png)
### Importing and Defining 
_______________________________
Next, we will head over to our `app.py` in order to import the required xray components to start setting up our app to send telemetry to our AWS accounts xray service. We do this by `cd`-ing into our `backend-flask` dir then including this line of text:
```
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='Cruddur', dynamic_naming=xray_url)
XRayMiddleware(app, xray_recorder)
```
![Untitled4](https://user-images.githubusercontent.com/114888726/221945410-1253b42c-cbff-4c0c-a096-cb0828e3232e.png)
**NOTE** The position of each line of code should be placed exactly as presented on the screenshot. I highlighted each block of code in the screenshot to show where exactly to place the code block provided above. 

Now that we have imported nad defined xray in our `app.py`, we need to define a JSON file that will allow us to link our app to xray. First, make sure you are in the `aws-bootcamp-cruddur-2023` directory before running this mkdir command: `mkdir -p aws/json`. The `-p` will create a parent directory and any sequentially ordered sub directories within that parent. After you've created the folders, `cd` into them ensuring you are in the `json` sub dir. Then you will `touch xray.json` within that sub dir with the following content inside that file: 
```
{
  "SamplingRule": {
      "RuleName": "Cruddur",
      "ResourceARN": "*",
      "Priority": 9000,
      "FixedRate": 0.1,
      "ReservoirSize": 5,
      "ServiceName": "Cruddur",
      "ServiceType": "*",
      "Host": "*",
      "HTTPMethod": "*",
      "URLPath": "*",
      "Version": 1
  }
}
```
![Untitled3](https://user-images.githubusercontent.com/114888726/221947260-21d7f74c-6218-4803-96d4-dec2626a5e7b.png)
It will loook something like this. At this point, we will use our awscli (which should be defined and installed within your workspace through your `gitpod.yml`) to create a "Cruddur" group for xray to trace. The commands are:
```
aws xray create-group \
   --group-name "Cruddur" \
   --filter-expression "service(\"backend-flask\")"
```
You will receive a JSON style output on your terminal if you ran this command succesfully. ![Untitleda](https://user-images.githubusercontent.com/114888726/221948967-2f31f7d4-b393-4926-a122-c9f5ebd0807a.png). If you head over to your AWS management console and make your way to the xray service, if you click groups, you should see the "Cruddur" group we just finished creating.  ![xray_trace_groups](https://user-images.githubusercontent.com/114888726/221952592-73e56083-b063-49ca-bbb0-b00ff21c3642.png)

Now we will have to create a sampling rule for our traced group. These rules allow us to place specific parameters for tracking purposes. If we don't do this, you will be be asking the xray service to keep track of many rules which can run that 100,000 traces quickly thus incurring charges. The command and sample rule we will be defining will be entered into our gitpod cli:
`aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json`
You will get back some output on your terminal in JSON again, if you did each step correctly thus far: ![sample_rules_terminal](https://user-images.githubusercontent.com/114888726/221954458-7bf75f09-0392-48a2-9643-d8883c9b68af.png)
Navigate back to your AWS management console and these sample rules should also be created under the "sampling rules" tab.
![sample_rules1](https://user-images.githubusercontent.com/114888726/221952996-42cca152-1130-4336-a481-8a4241c812c2.png)
We are almost done, now we need to set up a container to host the xray daemon that will be tracing our application. 

### Docker time!
___________________________
Now, in order for this to work we need to ensure this works we have to do three things: 1) To include the xray daemon into our `docker-compose.yml` to track our cluster of containers & 2) Define the environment variables that will define the endpoint the xray daemon will be extrapolating the trace info from as well as the AWS region we're working from and 3) Create a new env var in our gitpod workspace that will host the AWS Region we are working out from.

**STEP 1**
```
  xray-daemon:
    image: "amazon/aws-xray-daemon"
    environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "us-east-1"
    command:
      - "xray -o -b xray-daemon:2000"
    ports:
      - 2000:2000/udp
```
![Untitled](https://user-images.githubusercontent.com/114888726/221954664-3dc4fdaa-edef-46d5-ac86-4a7a3e9e1228.png)

**STEP 2**
```
      AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
      AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
```

![Untitled5](https://user-images.githubusercontent.com/114888726/221955904-f11daaf5-9aa4-4eed-9447-2d603c037c53.png)

**STEP 3**
**Remember** This step will be done in your gitpod CLI on the command prompt **NOT** in the `docker-compose.yml`. 
```
export AWS_REGION="[YOUR_AWSREGION]"
``` 
```
gp env AWS_REGION="[YOUR_AWSREGION]"
```
Once you've done all of this, we can prop up our containers with the `docker compose up -d` command. If done correctly, you will see some trace activity from Xray when you attempt to connect to the backend. It is highly suggested you connect to the URL provided by gitpod in the `ports` tab, and append `/api/activities/home`. This will take you to the API page that is hosting the backend for our `home_activities.py`. Refresh it a few times then head over to your AWS management console, make sure to get to the xray service using the search bar provided by AWS then go to traces. If you don't have anything appear, try pressing the orange button 'Run query" and you should see some data appear on your chart like the example below.  
![Untitled6](https://user-images.githubusercontent.com/114888726/222277569-0e89eb5e-1b73-420c-8c3f-852e6602753f.png)
**Suggestion** Go over to your honeycomb to make sure your app is still sending telemetry to it. 
![Untitled7a](https://user-images.githubusercontent.com/114888726/222280868-bd2282e7-3607-47f2-aeb4-d2db24be718f.png)

### Tracing specific services
______________
Now that we have a general trace on our application, lets add a handler into our `user_activities.py` in order to receive telemetry specifically from it. To do this we need to define it within the file of the same name. Before adding any new lines, please be sure to `docker compose down`. The lines you will include are: **Note** This will contain several lines of code that have mandatory placement within this file. Refer to the screen shot provided in order to locate these lines.
```
from aws_xray_sdk.core import xray_recorder
segment = xray_recorder.begin_segment('segment_name')
dict = {
      "now": now.isoformat()
    }
    subsegment = xray_recorder.begin_subsegment('mock-data')
    subsegment.put_metadata('key', dict, 'namespace')
```
**Note** Be mindful of the identation as it can effectively break your code if not done correctly. Refer to the screenshot below for how the indentations should go. 
![Untitled7](https://user-images.githubusercontent.com/114888726/222281810-7181bcb5-498a-4cb4-8e8f-823b4f404daa.png)
After this, redeploy your containers and attempt to connect to the webpage for the backend. If it works you'll get the 404 error. If you received it, head over to your management console one more time and you will see the activity appear on graph. Lets move on to cloudwatch logs. 
![Untitled9](https://user-images.githubusercontent.com/114888726/222282236-6844f89e-e668-476a-a554-15228c871c04.png)

## Cloudwatching
AWS Cloudwatch logs allow us to log our application's processes. In order to integrate this service onto our app, we are required to add a new requirement to our `requirements.txt`, that being `watchtower`. Watchtower is the log handler for cloudwatch logs. Make sure that after you add it to your `requirements.txt` that you also run the command for our favorite python package manager, `pip install -r requirements.txt`. As always **make sure you are in the `/backend-flask` folder before running this command**. ![Untitled1](https://user-images.githubusercontent.com/114888726/222283078-6f72262b-f825-4a39-9d25-748ab7194d9c.png)

Next, navigate over to your `app.py` so we can import the `watchtower` handler along with the following code: 
```
import watchtower, logging

LOGGER = logging.getLogger(__name__)
LOGGER.setLevel(logging.DEBUG)
console_handler = logging.StreamHandler()
cw_handler = watchtower.CloudWatchLogHandler(log_group='cruddur')
LOGGER.addHandler(console_handler)
LOGGER.addHandler(cw_handler)
LOGGER.info("some message test")
```
![Untitled2](https://user-images.githubusercontent.com/114888726/222283549-f31f28f9-30d0-4290-9617-4cd626a6e2ea.png)
Alright, the next thing we need to do is to add a handler for any type of errors. This will be placed at the bottom portion of our `app.py`, near to where we defined our APIs. You will add these line of codes to the file:
```
@app.after_request
def after_request(response):
    timestamp = strftime('[%Y-%b-%d %H:%M]')
    LOGGER.error('%s %s %s %s %s %s', timestamp, request.remote_addr, request.method, request.scheme, request.full_path, response.status)
    return response
```
**Note** This must be included in the section presented in the screen cap, otherwise the error handler will fail to function and break your backend. 
We also need to do two other things in order to ensure we get the logs sent to our cloudwatch logs: The 1st thing is to head over to your `app.py` file, drop to the 120th line (might differ between users but should be approximately in the same space), and where our api is for `"api/activities/home"`, we will redefine our function in order to ensure that the other code will be adding to `home_activities.py` is picked up by the api. The 2nd thing is to then head over to that `home_activities.py` and redefine the `HomeActivities` class. 
For the 1st step the code is:
```
@app.route("/api/activities/home", methods=['GET'])
def data_home():
  data = HomeActivities.run(logger=LOGGER)
  return data, 200
```
For the 2nd step the code is:
```
class HomeActivities:
  def run(logger):
    logger.info("HomeActivities")
```
![Untitled7](https://user-images.githubusercontent.com/114888726/222286677-8c4c4abc-2106-4118-884e-ba2e1f6cedc4.png)
**Note** Some of these lines already appear within both `.py` files respectively, so please be aware that when you are redefining this lines of code, to ensure you are not doubling up on whats already there. All of these lines should only appear once within the files, in the respective places seen on the screenshot. 
Lastly, we will need to define the credentials for AWS account within our `docker-compose.yml` using the environment variables we used before. Those being 
```
AWS_DEFAULT_REGION: "${AWS_DEFAULT_REGION}"
AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
```
Refer to the screenshot for reference on line positioning of the code. ![Untitled6](https://user-images.githubusercontent.com/114888726/222287435-55a5e6d5-4e5a-4879-ad12-fa8e8f472c96.png)
We can now run `docker compose up -d` to prop up our cluster. We will head over to our AWS management console, navigate to `log groups` and select our 'cruddur' log group. From there you should be able to view the sample text we included in our `LOGGER.info()` in the `app.py` file. I have marked it on the screenshot so you can view the log that captured it. ![Untitled8](https://user-images.githubusercontent.com/114888726/222288023-4ca15fd0-828a-4577-bd11-a9a2aa29e1cb.png)
Voila we are done with setting up cloudwatch logs! Lets move on to rollbar.

## Rollbar-ing
Rollbar is an Error logging and tracking service for developers. It allows us to track bugs and monitor for solutions on our systems. If you haven't already make sure to create a Rollbar account in order for you to actually use the service and integrate it into your application. Once signed in, you will be required to choose the framework you are utilizing define your backend. In this instance, you will select 'flask'. Once you've done so it will being you to an instructions page which has steps and sub-steps you need to take in order to integrate Rollbar. However, before we do any of that there are a few things we 'require' in order to get this going.

The first thing that is needed will be some new requirements in our `requirements.txt`, that being `blinker` and `rollbar`. 'Blinker' is a fast dispatching system (essentially a library that can be used to send info faster) that allows any number of interested individuals to subscribe to events that occur within our application. Make sure to do this in our `/backend-flask/` and run the `pip install -r requirements.txt`. ![1](https://user-images.githubusercontent.com/114888726/222289779-ad120639-66b9-4809-9923-6c3c925fec65.png)
![2](https://user-images.githubusercontent.com/114888726/222289785-fdd18dfe-f958-40eb-baad-a39dcd42c113.png)
Once the dependencies are installed into our environment, we will go ahead and return back to the instructions page on Rollbar's website in order to collect our rollbar token which can be found in 'Step 4' (left hand task bar) under 'Step 2 of 4'. There you will find the token inside of a markdown box to copy your token from. ![3](https://user-images.githubusercontent.com/114888726/222290103-8ce9465c-e1ff-43cc-aeee-31ef4df1936d.png) We need to head back over to our gitpod workspace in order to define the environment variable that we will refer to in our code so that the credentials aren't stored within our application. Remember to do this we need to use both the `export` and `gp env` command to set these within our shell and our gitpod account. 
```
export ROLLBAR_ACCESS_TOKEN=""
gp env ROLLBAR_ACCESS_TOKEN=""
```
Then run the `gp env` command on your command prompt and it should appear with the rest of the env vars we've defined thus far. ![4](https://user-images.githubusercontent.com/114888726/222290544-6266550a-e9e7-41c8-8f12-f93f517bd1bf.png)
**Note** The '""' are empty strings that require your token, so make sure you paste it inside the double quotation marks. 

Now, will have to import and define the code that will be responsible for getting rollbar to work. `cd` into our `backend-flask` and head to `app.py`. You are going to enter this:
```
import rollbar
import rollbar.contrib.flask
from flask import got_request_exception
```
![4](https://user-images.githubusercontent.com/114888726/222290956-95b3cf81-b5bc-4a1d-93f5-85117a56c9b6.png)
Then you will scroll down and add this beneath our where our code for the flask app begins, this appears as `app = flask(__name__)`. This is the code you will add:
```
rollbar_access_token = os.getenv('ROLLBAR_ACCESS_TOKEN')
@app.before_first_request
def init_rollbar():
    """init rollbar module"""
    rollbar.init(
        # access token
        rollbar_access_token,
        # environment name
        'production',
        # server root directory, makes tracebacks prettier
        root=os.path.dirname(os.path.realpath(__file__)),
        # flask already sets up logging
        allow_logging_basic_config=False)

    # send exceptions from `app` to rollbar, using flask's signal system.
    got_request_exception.connect(rollbar.contrib.flask.report_exception, app)
```
![6](https://user-images.githubusercontent.com/114888726/222291859-c01dd4cd-f928-4364-9901-f8461846cd61.png)
**Note** The position here is key so make sure to place it exactly as it appears on the screenshot.
We are almost there! We have to add an endpoint in order for rollbar to receive our error test message. This will be included below to where the APIs are:
```
@app.route('/rollbar/test')
def rollbar_test():
    rollbar.report_message('Hello World!', 'warning')
    return "Hello World!"
```
![7](https://user-images.githubusercontent.com/114888726/222292152-01c71584-7f27-45e4-80e8-24092f813cf8.png)
The final thing we will have to do here is add another environment variable within our `docker-compose.yml` in order for the credentials we set into our environment to be utilized within our containers. ![7a](https://user-images.githubusercontent.com/114888726/222292436-be9bc878-5cfb-4932-8840-6b1daa92da8b.png)
Now, lets `docker compose up -d`, prop up those containers and lets head back over to our rollbar webpage to see if it received the error message.
![7b](https://user-images.githubusercontent.com/114888726/222292560-75f900b0-c630-4e65-8d7f-7cd97665a75f.png)
As you can see it did! You can also test this by connecting to your backends url and appending to that url : `/rollbar/test`, and the "Hello World!" message should appear on your screen. And thats it we have integrated rollbar to capture errors. However, we need to include end points for our services so this is an extra step required to fully take advantage of the service. Lets move on to the only challenge homework I was able to get to this week, creating a custom query in honeycomb. 

## Homework Challenge - Custom Queries
