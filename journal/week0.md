# Week 0 — Billing and Architecture
## Spend & Security Concepts
I started week 0 by viewing Chirag's spend and Ashish's Security consideration videos in order to refamiliarize myself with navigation through the management console. Specifically, to review how to set up a budget, alarm, setting up the SNS for the billing alarm and setting up IAM and MFA. Both videos are very informative so I suggest my fellow cohort to review these videos as well.  

Since I already have an AWS account with an IAM user and root both set up with MFA I skipped this step. However, I did remove admin policy privileges that I had attached to my IAM user and instead created an "admin group" through the management console with the same admin policy attached to said group. Afterward I moved on to setting up my budget and billing alerts. 

Although Andrew decide to demonstrate how to do this on the CLI. I did not utilize this resource as I had already watched the previously mentioned videos which taught me how to set up both budget and billing alerts so I found it unecessary to delete what I did just to have practice on the CLI. Anyway, I followed along to Chirag's instructions as I navigated my management console. I did this through my Root account as I did not find it necessary to provide my IAM User access to my billing account or creating an entirely separate account to access billing as jumping between accounts seemed like a very inefficient way to use up my time. Regardless, once I was logged into my root account, I navigated the management console and went to "Billing Preferences". Here, Chirag explained it was wise to mark off two check boxes found under "Cost Management Preferences": Receive Free Tier Usage Alerts & Receive Billing Alerts. Both would respectively, inform me when my Free Tier Usage was approaching or exceeding the limit for that month via email notification & in order to receive notifications when thresholds set within my billing alerts and budgets were also approaching or exceeding the set budget. 

Next, I went CloudWatch, a service which tracks service usage where you can set alerts based on pre-defined or custom defined metrics that you can create. I navigated to this page by using that search bar within my web browser window. Once I was on the page (and as per Chirag's guidance), looked to the side task bar and navigated to "All alarms". If you haven't set up any alarms yet then the dashboard should be blank and requesting that you create one. I did exactly that by pressing the orange button labeled "Create alarm". I selected the "Billing" then "Total Estimated Charge" metric. This was I can cast a large net by which to monitor all possible usage within my account. It will also ask you to select your currency, (this is subject to change based on the region you're working from)since I am in the USA, I selected USD. It will then forward you to a different page where you can set the conditions that will define your thershold, I choose a "Static" thershold type whenever the estimatecharges are "Greater" than the limit you set. This limit represent your overall bottom line and entirely depends on how much you want to spend on services within AWS. Obviously, we are trying to mitigate all cost but as I am willing to spend at least $10 dollars in total for the duration of my time within the cloud camp (and thanks to the free credits I got from Andrew as well as AWS due to a billing error I received outside of this bootcamp), I set my limit to $10 USD. 

Now, once you've went to the next page, you can set up your alarm trigger by creating a new topic or using a prexisting one. Once again, under the impression you've just created this account, you would go a head and create a new topic. You can name the topic and send an email endpoint as to receive these notifications to the specified email address you enter. After this page, you'll be taken to the final page to review everything you've entered. If all looks well then you can finalize it by clicking that orange button you've been doing to move to the next sequence of setting up the alarm and voila, you have yourself your Billing Alert. Be mindful that, despite using free tier if you push past the set limit for email notifications (which according to sns pricing is 1,000 notifications)you will see a charge of $2.00 per 100k notifications. It would probably take a lot of mistakes on your end to have services running for the alarm to consistently be set off this way, but since we are being guided through the process of creating an application and deploying it through cloud infrastructure, we should be avoiding this charge entirely. 

Last thing before moving on to setting up gitpod, we need to create a budget for this project. To do this, we are gonna use are handy dandy search bar to look up "AWS Budgets". Once you've clicked it, it will take you to the empty dashboard asking you to create some budgets. Now as this point, I deleted what budgets I created with Chirag since the aforementioned video Andrew posted up got very granular on how we should set up our budgets for this bootcamp. He suggested in his video that we create two budgets: one to track actual dollars and the other to track credits we received. According to AWS Budgets Pricing, your first two action enabled budgets are free regardless of the number of actions you've configured per budget, so do not exceed this limit unless you want a daily accruement of $0.10. Anywho, on this blank dashboard will go a head and click "Create a budget". 

On the following screen, we can set up one of many parameters to hav ebudgets alert us on a daily, monthly or custom period regarding spend. As per Andrew's instructions, we will go a head and select "Zero Spend Budget". We will provide it a name appropriate to its use then provide an email for notifications to be sent too. Now, in order to respectively define both Dollar and Credit budget, we will have to edit both so they can reflect the exact type of expenditure both will be tracking. For my dollar budget I made it Monthly for the "Period" with an "expiring budget" that will range from this month till June 2023. I left the "Budgeting method" as Fixed, and set the dollar amount to $10 (but this can vary for you). I left the "Budget Scope" on the recommended setting and left "Aggregate costs" to "Unblended costs". We will proceed to press "next" at the bottom of the screen and set the Alert. Since it is a Zero-budgeted amount, the lowest you can go is "0.01" but upon creation and viewing on the budget's dashboard, it will be demarcated as "$1.00". You can set the "Trigger" for "Actual" or "Forecasted". I added an additional Alert for a percentage of actual spend, so I can be alerted if my budget ever started to rise. You can proceed to hit next until you're allowed to save your configurations and will be returned to the budgets dashboard. You will then follow the same steps to create a "credit" budget, but with one exception. Where "Aggregate costs" is on the bottom of the page, deselect all the default options and click "supported charge types". Here you can deselect or select options you want tracked. You will find the "Credits" option here. I also put "Refunds" in case I ever incur a charge but speak to AWS's Support Team for a refund, IF it was incurred in error. Now lets set up our gitpod!

## Creating AWS Credentials
Alrighty, this was pretty fun for me. I followed along to Andrew's video "Week 0 - Generate Credentials, AWS CLI, Budget and Billing Alarm via CLI". I woudl suggest you to do the same if you have no experience with gitpod or setting up your AWS CLI on an IDE. Now the first thing you have to do, is acquire an Access Key and Secret Key from AWS itself. To do this will log into our IAM User that we created with Admin privileges. Now, on the upper right hand corner you will find your IAM User name, click that and a drop down menu will appear. This menu will have a "Security Credentials" option to select, so please do so. Once you're here, scroll down to find the option to "Create access key". When you do this, you'll be taken to a screen with the CLI option as the first pick, click that and proceed. 

#### **Disclaimer: Two things, one: there will be a warning from AWS asking you use Cloudshell (an in browser CLI to programmatically interact with AWS Services) or to use the AWS CLI V2 and enable authentication in IAM. You can check the box off and appreciate how much AWS cares about you not getting hacked!** 

It will ask you to create a tag, which you should label appropriately and click next one last time so you can view your Access key and your secret key. **Please DO NOT SHARE YOUR KEYS WITH ANYBODY!** Doing so will allow individuals to muck up your account and that would be unfortunate. Now with a generated key we can jump into our gitpod account and set up our workspace!

## Setting up Gitpod workspace and AWS CLI

So to append to what I said on the last chapter, this was another fun part for me. I had already set up my gitpod and linked it to my github along with providing it access to the template Andrew created for us which can be found on his github profile under "aws-bootcamp-cruddur-2023". There should be a big green button that says: "Use this template". Andrew has a video explaining how to appropriately set this up and can be found on his youtube channel with the title: "Creating Your Repository from the Github Template". Moving on, Once you are logged into your account, there will be a .gitpod.yml file within your directories. We are going to set up the environment using this yml file. This way, everytime we start up our gitpod environment, it will basically be scripted to set up the environment so we can always have access to AWS CLI. Andrew goes over how that yml will look like on his "Week 0 - Generate Credentials, AWS CLI, Budget and Billing Alarm via CLI" video but I am gonna post it here anyway: 

```
tasks:
  - name: aws-cli 
    env:
      AWS_CLI_AUTO_PROMPT: on-partial
    init: | 
      cd /workspace
      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      unzip "awscliv2.zip"
      sudo ./aws/install
      cd $THEIA_WORKSPACE_ROOT
vscode:
  extensions:
    - 42Crunch.vscode-openapi
```

#### I would suggest testing this out by opening a new window to gitpod, re-entering your workspace and watch the test fly by so quickly on the terminal tab underneath that you won't know what happened (its just the yaml file doing its thing ;D). To test out if it worked, just type in "aws" on the terminal and this should appear:

```
usage: aws [options] <command> <subcommand> [<subcommand> ...] [parameters]
To see help text, you can run:

  aws help
  aws <command> help
  aws <command> <subcommand> help

aws: error: the following arguments are required: command
```

Alright this is the very last thing we need to do and we should be all set! We will have to create environment variables in order to store our AWS credentials. This way everytime we get on gitpod and utilize the AWS CLI, it will automatically have the variables with the necessary access key and secret key to have permissions to use resources on our account. To do this we have to use a command: `gp env`. In combination with the names of the variables we will assign our credentials and regions into. This information can befound on the video but also on AWS. The commands and variables you'll enter into the terminal will look like this:

```
gitpod /workspace $ gp env AWS_ACCESS_KEY_ID="[Enter your access key here]

gitpod /workspace $ gp env AWS_SECRET_ACCESS_KEY="[Enter your secret key here]"

gitpod /workspace $ gp env AWS_DEFAULT_REGION="[Enter your region here]"
```

Once you've done this, open another tab to gitpod and click on your profile icon on the upper right hand corner of the screen and click "User Settings". Once the new page is loaded, you will see a option for "Variables" on the left hand side of the screen. When you click this, it should list the variables you set up for this specific workspace. The last thing will do is make sure that the AWS CLI is recognizing our credentials. We will do this by entering the following command: `aws sts get-caller-identity`. In response the terminal will display values in a JSON format that will look something like this: 

```
{
    "UserId": "[Access key ID]",
    "Account": "[account number]",
    "Arn": "arn:aws:iam::[IAM alias]:user/[IAM Username]"
}
```

Ladies and gentlemen, we did it! We are now ready for Week 1! If you have any issues, I would suggest heading to the discord to seek assistance from the moderators or Andrew. Beneath this will be the final chapter but it won't have much writing. It will display my two lucidcharts respectively representing my conceptual and logical architectural design for our application! Thanks for reading and I'll see you in the next episode!

## Lucidchart shared links

Conceptual Architectural diagram: https://lucid.app/lucidchart/61bf18e8-03e1-4db3-b498-ec97821e2dcd/edit?invitationId=inv_5d4a34ce-e5e5-44cd-bcf3-d01a724ff67d
Logical Architectural diagram: https://lucid.app/lucidchart/invitations/accept/inv_0dc02fba-730b-4516-99c1-8dfb61e272b6

I did not make many changes to the conceptual diagram as I wasn't sure what other considerations to include on the diagram itself. I tried to seek help on discord, but I was having issues at the time with verifying my phone number cause it "supposedly" was associated with another account despite never opening a different account with my newer phone number... As for the logical diagram, I tried to improve upon the original model by cleaning up a lot of the lines and making it more cohesive as the original diagram got a bit messy when linking applications to services like RDS, DynamoDB and AppSync. I even included a public and private subnet cause I figured we will be isolating these two containers as best practice since we don't want any bad actors to have an easy time accessing our backend which is connected to our DBs thus allowing access to customer info. I was going to also add NACLS and SGs but I liked how cleaned it looked and I figured since it wasn't on the document outline provided by Andrew, I'd leave it out until this was mentioned. 