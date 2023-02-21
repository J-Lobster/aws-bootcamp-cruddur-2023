# Week 1 — App Containerization
### Setting up Cruddur Notification service - **API**
In order to add "notifications as a service to Cruddur, we are first required to create endpoints to our two components, Front and Backend, using OpenAPI.
First, we need to install the OpenAPI Extension, via your Gitpod IDE. If you press `control + shift + x` while your in your Gitpod, it will take you to the extensions tab. Merely type in "OpenAPI" on the search bar above, it will be the first option that appears. Once installed we can get started on configuring the `openapi-3.0.yml` file, which can be found in `aws-bootcamp-cruddur-2023/backend-flask/`. 

![Open_API_searchbar](https://user-images.githubusercontent.com/114888726/220210469-287d0923-6200-4893-ab9a-549f77b96299.png)
**Note** I intentionally used VSCode as the UIs are exactly the same. The shortcut to the 'extensions' tab also works exactly the same, I made sure to test it. 

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
After you've done this, make sure to commit and push your code to your repository. Next up, will work on some docker challenges.

### Docker Challenges - `docker exec` CMD
When composing a Dockerfile to create an image, its common to use the `CMD` module within our file to order docker to run a command within the container upon running. A good example is to start up a httpd server. However, the `CMD` module in our Dockerfile, can also be used externally as a command on your CLI. This command is `docker exec`. It includes several optional flags and arguments but for this example I am going to execute a `cat` command within my container in order to view the Dockerfile that was copied from the working directory. 

For this example, I used an image I built out in the past. I deployed it using the `docker container start` command while appending the image ID to the end so it knows what image to use for deployment. I then used `docker ps`, to confirm that the container was up. Lastly, I ran the `docker exec [CONTAINER_ID] cat [FILENAME]`. This concatenated the file within the container, bash script: ![Exec_command_Execution](https://user-images.githubusercontent.com/114888726/220475194-81ec520a-5ac9-4cf6-a947-073e0f26d1fd.png)

### Docker Challenges - Tagging and pushing
For this challenge, I had to relearn how to use the `docker tag` & `docker push` commands, in order to add an image into my person repository, hosted on 'Docker Hub'. 'Docker Hub' is a centralized repo for container images. Users can sign up for the free tier but there is an enterprise version that allows you to make more than 1 private repository for use by buisnesses. The first step to pushing an image to your Docker Hub is to create an account on 'dockerhub.com'. The next step would be to install the Docker client via your CLI or web browser. You can find the Docker Desktop client on their website at 'https://www.docker.com/'. Once you've finished installing it, you will want to sign into your account through the Docker Desktop client. 

Now, before you do anything else. I implore you to create an access token to programmatically sign in and push images up to your docker hub account. It is much more secure to do it this way rather than to sign in with a password. First, navigate to your account settings and select security tab. From there, you will have the option to activate 'two factor authentication' & generate an access token. Click the button to create the access token and I implore you activate two factor authentication for security purposes. It is best practice afterall. 
![Navigating_to_security_settings_in_dockerhub](https://user-images.githubusercontent.com/114888726/220476905-3a409be2-b33d-410d-b217-89b1092255e6.png)
![Security_tab_on_dockerhub](https://user-images.githubusercontent.com/114888726/220476921-ae7dc568-9fea-4bee-83e5-e3fc054a1a25.png)
The next step, is to log into your docker through your CLI, using the `docker login -u [USERNAME]`. You will be promted to enter your access token, which you should have created and saved by now, enter that here and you'll be signed in. ![Successful_login](https://user-images.githubusercontent.com/114888726/220477304-e684d8fe-381c-4f7b-afe2-405abef06b47.png)
Now, take a preexisting image in your library or build a new one. We are going to use the `docker tag [IMAGEID/IMAGENAME] [USERNAME]/[IMAGENAME]:[VERSION]` command to label the image as our own. Without appending our username to the image, the docker daemon won't know where to push these images to and will default it to the 'docker.io' registry which you will be denied access too. Make sure you place the image id or name of image just before you place your username as well as the version number. Its best practice to tag your images by version, in order to maintain it and allow others to know when you have changed the image to a better version. ![bagem taggem](https://user-images.githubusercontent.com/114888726/220477979-8ec93dee-f499-4400-b557-aaa6d244f859.png)

Last thing, is to push the image up into our docker hub repository. `docker push [USERNAME]/[IMAGENAME]:[VERSION]` will allow you to then push your selected image into your repo. If done successfully, it should appear similarly to what I have posted here: ![Successful_Image_Push](https://user-images.githubusercontent.com/114888726/220478360-904e6f99-520b-450f-9a4b-649ce1f62d30.png)

## Docker Challenge - Multi-stage builds
In Docker, its best practice to strip away as many layers from an image as possible. These layers are made up of filesystems that capture the state of the image before using it as a base to transpose other layers onto the image until it is a full image. This process is what allows images to stay lightweight, make them reusable, and leaves your image and the container it creates much more airtight from security breaches. The problem starts when you have to create a single image from a single Dockerfile and then creating bash scripts (if linux) to automate the building of one image to another to create a small sized version. Thankyfull, docker has a feature called 'Multi-stage builds', which allow us to create multiple images using a Dockerfile to define both images before layering them together, thus creating a much lighter weight image and container. 

We create a multistage build by including an additional `FROM` within the Dockerfile itself. `FROM` tells docker daemon to pull a specific base by which to build out your image for your application. There is another keyword `as`, which will allow us to make a reference to a specific piece of the build through an alias. Best practice usually signifies the first image as the `Build` image, as it is what the next stage of the image will layer itself from, using the initial built image as a base. 
![Dockerfile_multistage_build](https://user-images.githubusercontent.com/114888726/220480845-2e71096a-0b24-4727-a564-69a8e0705837.png)


Thanks to Lukonde Mwila for his repo "builder-pattern-example" for providing a js and react pages in order to test out this container built through multi-stage. Anyway, you will now build out the image using `docker build -t [IMAGENAME]:[VERSION]` and you should see docker display some output on your terminal. You can see that before building out each image, it goes through literal 'stages' as it steps through the build layers. ![creating a multi-stage build](https://user-images.githubusercontent.com/114888726/220480355-d63034f3-d04b-4e9e-a388-20579d0874e8.png)
We will list out the images we have and create the container using the commands on this screen shot: 
![mutli-stage container built](https://user-images.githubusercontent.com/114888726/220480510-bd4c93fa-cfe2-42e6-b176-c6f09445292b.png)
Then, we will try to connect to our nginx server to see if our react page is working. If it does, you will receive a different message from me as I went into the .css file and changed the text to doubly make sure everything was done correctly: ![connected to multi-stage container app](https://user-images.githubusercontent.com/114888726/220480753-293b8869-32e1-47aa-b82b-ed8c5accbabe.png)

## Docker Challenges - `HEALTHCHECK`
Another service provided by Docker are `HEALTHCHECKS`. These can be used either in a Dockerfile or a docker-compose.yml. These checks will monitor the health status of a container. We can set it for interval checks, to time out after a certain period of non responsiveness and even set it so that other containers will only initialize upon creation. Whether healthy or unhealthy both statuses will appear if whenver you list out your running containers. In this example, will be using a docker-compose file to deploy a postgres server using Kong as its API for data migration between the two containers. Peter Evan's has a great repo with a detail explanation on the use of `HEALTHCHECK` for this particular use case and can be found on github under "docker-compose-healthcheck". The image below displays the compose file within my CLI.
![reviewing healthcheck syntax](https://user-images.githubusercontent.com/114888726/220481598-ac4f3925-3fd9-4177-a197-0bb03acfd8be.png)
As you can view, the `HEALTHCHECK` will send a command using `test:`, this command is unique to node.js and will run the healthcheck to ensure postgres is ready and functioning properly. If you look at `kong-migration` and `kong` services, you can see underneath `depends_on`, that both require that the postgres (kong-database) is in a serviceable state before they build themselves out. ![Service status dependency](https://user-images.githubusercontent.com/114888726/220482205-8957faab-3adf-4579-866a-e2706bfb5387.png)
Now will `docker compose up -d` the docker-compose.yml and we can see by the end of the image build, that the `container kong-postgres` is in a 'healthy' state. This will be the second thing to resolve as the `Network` is a separate service that will allow these containers to communicate to one another in their own network. ![image_pull_build_ healthcheck_status](https://user-images.githubusercontent.com/114888726/220482439-9725af45-1a84-4c54-9b8f-88f3816f7f77.png)

We will do one final `docker ps` or `docker container ls` to view the status of our containers which should say 'healthy':![final_healthcheck](https://user-images.githubusercontent.com/114888726/220482512-01890c81-2baa-4fef-92db-5cdfcdf36855.png)

### Docker Challenge - Security
I watched Ashish's video container best practices and learned about a few 3rd party tools available to docker to check for vulnerabilities within our containers. A few of the tools provided were: Amazon ECS, Amazon Inspector, Snyk and clair. I ended up using Snyk to check the vulnerabilities of our cruddur application and see what we can improve in order to maintain our container's security. First you'll want to create an account with Snyk. We can use Github or Google as federated users, I used github. This allowed me to pull a repo from my account so I used the "aws-bootcamp-cruddur-2023", which makes it easy to configure what repos we want to scan or not.![using_snyk_to_scan_cloudcamp_repo](https://user-images.githubusercontent.com/114888726/220483617-7e1ad539-e264-465b-a770-627199a5da5b.png)
We will then navigate to projects and we will be presented with our Dockerfiles as well as our .json packages and code. Snyk allows us to scan all of these but we are only checking our container images for now. ![Repo_Dockerfiles_vulnerabilities](https://user-images.githubusercontent.com/114888726/220483826-e67bdb08-a68b-4736-8a01-88d95ba91769.png)
Select either of the images, where it will forward you to another page that list the vulnerabilities specific to that Dockerfile. You will see a button on the right hand side called 'Open a fix PR'. This will allow Snyk to go in and fix up the vulnerabilties to your code by updating the Dockerfile with the latest images on the `FROM` line. It will make a pull request to your github, in order to include the new fixes to your Dockerfiles and remove current vulnerabilities. ![repo_dockerfiles_granularcheck](https://user-images.githubusercontent.com/114888726/220484448-cb1a3faa-1a13-4201-a42f-4d42171f89bf.png)
![snyk_vulnerabilities_fix](https://user-images.githubusercontent.com/114888726/220484457-4ca6ca54-9901-4ad8-8595-d7cf052e71dd.png)
As you can see, I decided to decline the pull request, merely because I don't want to include anything that is not instructed by Andrew so I can have an app that is actually working. Regardless, this is the end of what I did this week. I am looking forward to Week 2! 

**Note** I already had docker desktop client installed and had attempted to get the code to work on my machine but I ran out of time I allocate for myself during the week for Cloud Camp. I also would've hosted a container on an EC2 but the problem is I already exceeded my hours for EC2 for February. I got the credits back but the support team couldn't refresh the time for this month, hence, I neglected to do this final challenge. 
 
