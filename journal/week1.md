# Week 1 — App Containerization
### Setting up Cruddur Notification service - **API**
In order to add "notifications as a service to Cruddur, we are first required to create endpoints to our two components, Front and Backend, using OpenAPI.
First, we need to install the OpenAPI Extension, via your Gitpod IDE. If you press `control + shift + x` while your in your Gitpod, it will take you to the extensions tab. Merely type in "OpenAPI" on the search bar above, it will be the first option that appears. Once installed we can get started on configuring the `openapi-3.0.yml` file, which can be found in `aws-bootcamp-cruddur-2023/backend-flask/`. 

![Open_API_searchbar](https://user-images.githubusercontent.com/114888726/220210469-287d0923-6200-4893-ab9a-549f77b96299.png)
*Note* I intentionally used VSCode as the UIs are exactly the same. The shortcut to the 'extensions' tab also works exactly the same, I made sure to test it. 

First we gotta select the `openapi-3.0.yml` as we mentioned earlier. Then, navigate to the OpenAPI extension located at left side on your workspace task bar. It will appear like a small browser with the "/API" written within. We will open the file from within that extension, then navigate to the 151st line. There will be `...` on the `PATHS` portion of the OpenAPI extension, right click that and two options will appear. We are gonna select 'OpenAPI: add new path'.

![OpenAPI_creatingapath](https://user-images.githubusercontent.com/114888726/220211006-6cc83375-2b36-4f39-aaf8-1b2bd49a41a4.png)

Now, when you selected to add a path, what you really did was add a new line to write out yml required to set up the api for our notifications service. It will appear as: 

```
/name:               
  get:
```  

You're going to highlight that text then you're going to include the following lines: 
```
  /api/activities/notifications:
    get:
      description: 'Return feed activity for users you follow'
      tags:
        - activities
      parameters: []
      responses:
        '200':
          description: Returns an array of activities
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Activity'
```
This will allow our API to track activities related to our notifications service. We will be testing out this API later on but first we need to continue to connect our services together. Lets get started on the `app.py` file, which can be found in our `backend-flask` directory. 

### Setting up Cruddur Notification service - **App.py**
Once in this file, we will import from another service will be writing out in our `services` directory in a bit. This service will operate as our notifications program for cruddur. The first bit of text will be added on the 7th line, `from services.notifications_activities import *. We will then add a route and add a function that will return notification latent data.
```
@app.route("/api/activities/notifications", methods=['GET'])
def data_notifications():
  data = NotificationsActivities.run()
  return data, 200
```
![import_notifications_activitites_py_service-app_py](https://user-images.githubusercontent.com/114888726/220213502-649433f7-6468-4367-a33b-8474dd173b28.png)

![app_py_notifications_activity](https://user-images.githubusercontent.com/114888726/220213531-858e9ae7-aeb8-45a8-8708-ad30f5c64336.png)

For this final bit, we will create the **actual** `notifications_activities.py` that will service as our program to collect notifications sent through the API when users write out to users. Navigate to the `services` directory and add a new file named `notifications_activities.py` and add the following lines:

```
from datetime import datetime, timedelta, timezone
class NotificationsActivities:
  def run():
    now = datetime.now(timezone.utc).astimezone()
    results = [{
      'uuid': '68f126b0-1ceb-4a33-88be-d90fa7109eee',
      'handle':  'somegal',
      'message': 'Wow, I love Velma!',
      'created_at': (now - timedelta(days=2)).isoformat(),
      'expires_at': (now + timedelta(days=5)).isoformat(),
      'likes_count': 5,
      'replies_count': 1,
      'reposts_count': 0,
      'replies': [{
        'uuid': '26e12864-1c26-5c3a-9658-97a10f8fea67',
        'reply_to_activity_uuid': '68f126b0-1ceb-4a33-88be-d90fa7109eee',
        'handle':  'someguy',
        'message': "You're insane!",
        'likes_count': 0,
        'replies_count': 0,
        'reposts_count': 0,
        'created_at': (now - timedelta(days=2)).isoformat()
      }],
    },
    ]
    return results
```
Now, that import code will work within our `app.py` file since we just finished creating the
`notifications_activities.py` it is invoking. We can then test out this is working by launching a container and receiving a *`404`* error when attempting to connect to it. Also, make sure to `git commit` and `git push`, in order to commit and merge our code to the main branch. Up next, will set up our frontend and create a web page for our notifications service.

### Setting up Cruddur Notification service - **Frontend**


