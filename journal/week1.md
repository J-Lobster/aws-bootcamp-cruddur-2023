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

### Setting up Cruddur Notification service - **app.py**
Once in this file, we will import from another service will be writing out in our `services` directory in a bit. This service will operate as our notifications program for cruddur. The first bit of text will be added on the 7th line, `from services.notifications_activities import *`. We will then add a route and add a function that will return notification latent data.
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
Now, in order for the frontend to integrate with our newly defined backend code, will need to modify our existing `App.js`, which is in the `aws-bootcamp-cruddur-2023/frontend-react-js/src`. Within this folder, we will go to the `App.js` file and add a new line of code on line 4. This code will be `import NotificationsFeedPage from './pages/NotificationsFeedPage';`:![Modded_app_js_notificationsfeedpage](https://user-images.githubusercontent.com/114888726/220378732-38d371f7-36aa-4927-87f2-0db61b4e4f02.png)


Including one more line of code of line 24:
```
  {
    path: "/notifications",
    element: <NotificationsFeedPage />
  },
  ```
  ![included_path element_notificationsfeedpage](https://user-images.githubusercontent.com/114888726/220378796-b3d04394-a630-4836-85f2-628a6c001110.png)
Lastly, will need to create the actual .js and .css pages in order for the "Notifications" page to load in when we are logged into cruddur and navigate to the button which will take us there. The pages are found within the same `src` directory, that contains the `App.js` file. Make your way to the `pages` directory and create two new files named `NotificationsFeedPage.js` & `NotificationsFeedPage.css` respectively. From here, all we really have to do is fill in the `NotificationsFeedPage.js` with the right code. That code is:
```
import './NotificationsFeedPage.css';
import React from "react";

import DesktopNavigation  from '../components/DesktopNavigation';
import DesktopSidebar     from '../components/DesktopSidebar';
import ActivityFeed from '../components/ActivityFeed';
import ActivityForm from '../components/ActivityForm';
import ReplyForm from '../components/ReplyForm';

// [TODO] Authenication
import Cookies from 'js-cookie'

export default function NotificationsFeedPage() {
  const [activities, setActivities] = React.useState([]);
  const [popped, setPopped] = React.useState(false);
  const [poppedReply, setPoppedReply] = React.useState(false);
  const [replyActivity, setReplyActivity] = React.useState({});
  const [user, setUser] = React.useState(null);
  const dataFetchedRef = React.useRef(false);

  const loadData = async () => {
    try {
      const backend_url = `${process.env.REACT_APP_BACKEND_URL}/api/activities/notifications`
      const res = await fetch(backend_url, {
        method: "GET"
      });
      let resJson = await res.json();
      if (res.status === 200) {
        setActivities(resJson)
      } else {
        console.log(res)
      }
    } catch (err) {
      console.log(err);
    }
  };

  const checkAuth = async () => {
    console.log('checkAuth')
    // [TODO] Authenication
    if (Cookies.get('user.logged_in')) {
      setUser({
        display_name: Cookies.get('user.name'),
        handle: Cookies.get('user.username')
      })
    }
  };

  React.useEffect(()=>{
    //prevents double call
    if (dataFetchedRef.current) return;
    dataFetchedRef.current = true;

    loadData();
    checkAuth();
  }, [])

  return (
    <article>
      <DesktopNavigation user={user} active={'notifications'} setPopped={setPopped} />
      <div className='content'>
        <ActivityForm  
          popped={popped}
          setPopped={setPopped} 
          setActivities={setActivities} 
        />
        <ReplyForm 
          activity={replyActivity} 
          popped={poppedReply} 
          setPopped={setPoppedReply} 
          setActivities={setActivities} 
          activities={activities} 
        />
        <ActivityFeed 
          title="Notifications" 
          setReplyActivity={setReplyActivity} 
          setPopped={setPoppedReply} 
          activities={activities} 
        />
      </div>
      <DesktopSidebar user={user} />
    </article>
  );
}
```
![js css_created_notifications_page](https://user-images.githubusercontent.com/114888726/220380186-b16bdb0c-12ef-4daf-b649-a8dfa1b6c01f.png)

We are going to test out the container to see if the contents within the .js page, will load up when we connect to cruddur. First we will use our docker-compose file to set up the containers with `docker compose up -d` (make sure you are in the `/workspace/aws-bootcamp-cruddur-2023` directory, as that is where the compose file is). Once the containers are up, gitpod will prompt you to open ports and 'make public', be sure to do that. Once opened, head to the 'port' tab found by your terminal and click the url associated with port 3000. It will take you to the main cruddur page. You will sign in, which will require you make an account. Once it ask for email confirmation, you merely have to add your email address and the confirmation code is '1234'. Once logged in, on the left hand side where your task bar is, you will see "notifications" button, click it and it should display some message defined within the backend by the `notifications_activities.py` file we created earlier. I took some liberties with my own so do not fret if the messages and handler do not appear the same. ![notificationspage_Working](https://user-images.githubusercontent.com/114888726/220382091-ab5a90f6-c031-49e9-9d91-ac232066187d.png)
Make sure to git add and commit your code then push it to your main repo. This about wraps up frontend and backend set up for our notifications service. I am going to move onto setting up dynamodb and postgres DBs.

### Setting up DynamoDB & Postgres
To begin, the docker-compose file we've been working with has to include a few new services so when we deploy these containers, they will start with DBs associated to the system. Head to the `docker-compose.yml` file, scroll down to line 20 and make space to paste this code:
```
  dynamodb-local:
    # https://stackoverflow.com/questions/67533058/persist-local-dynamodb-data-in-volumes-lack-permission-unable-to-open-databa
    # We needed to add user:root to get this working.
    user: root
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
    image: "amazon/dynamodb-local:latest"
    container_name: dynamodb-local
    ports:
      - "8000:8000"
    volumes:
      - "./docker/dynamodb:/home/dynamodblocal/data"
    working_dir: /home/dynamodblocal
 ```
Next, we will also include our postgres DB just beneath the `dynamodb-local:` code block as well as create a volume to persist data captured within the postgresdb:
```
db:
    image: postgres:13-alpine
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    ports:
      - '5432:5432'
    volumes: 
      - db:/var/lib/postgresql/data
volumes:
  db:
    driver: local
```
**Note** The `Volumes:` module will not go in the same space as our `db` service. Instead, you will include this on line 49, just below the `networks:` module. Here is a screenshot as a better example of this: ![docker-compose_db_inclusion](https://user-images.githubusercontent.com/114888726/220388364-8632aca1-9439-40f7-babc-b811471d054c.png)

From here, I used `docker compose up -d` to deploy my containers and test to see if I can create a dynamodb table using the AWS CLI that should have booted up thanks to the `gitpod.yml` file. To do this the following the command can assist you to create a table:
```
aws dynamodb create-table \
    --endpoint-url http://localhost:8000 \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema AttributeName=Artist,KeyType=HASH AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --table-class STANDARD
```
If done successfully, you will get some output describing the table is JSON format:  ![Creating a table w dynamodb](https://user-images.githubusercontent.com/114888726/220390437-8911ab86-3708-43e1-81af-f29a8cb46891.png)

To further test this, lets also list out what tables are available to ensure everything is operable. Copy and paste this into your terminal next: `aws dynamodb list-tables --endpoint-url http://localhost:8000`. This should appear on your screen (**Note** Disregard that I have two tables, I created a second table since I had already created the `Music` table while I was following along to Andrew's video): ![Listing_available_tables](https://user-images.githubusercontent.com/114888726/220390967-76dd9db2-8799-456b-8ab9-1b860514ab5d.png)

Lastly, we will have to test the postgres db to ensure it is also functioning. Firstly, will need to install the postgres extension by cweijan. This will allow us to interact with the db with a simple UI that will appear on our workspace task bar on the left. Once downloaded, I would suggest defining this extension into your `gitpod.yml` file, in order for your workspace to start with the extension. We should also include installation of postgres CLI into the same file, so we can automate the set up. You'll want to add this on line 11: 
```
  - name: postgres
    init: |
      curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
      echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      sudo apt update
      sudo apt install -y postgresql-client-13 libpq-dev
```
I would suggest running the commands underneath the pipe "`|`", just so you have the DB installed onto your terminal. All together it should look like this:![updated_gitpod_yml](https://user-images.githubusercontent.com/114888726/220392897-985b58e8-f8c4-4216-8958-402734574188.png)

Now, for us to connect we have to run this command `psql -U postgres --host localhost`. You will be prompted for the user password, after entering the password, it should take you to a terminal with `postgres=#` as your command prompt. Type in `\t` and it should turn tuples on. ![Testing_postgres_operation](https://user-images.githubusercontent.com/114888726/220393680-5e617e1e-3aae-4937-816f-1345b7ee6339.png)


After you've done this, you are good to go with your Databases. Now lets move on to the docker challenges.




