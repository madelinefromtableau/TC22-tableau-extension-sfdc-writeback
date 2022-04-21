# TC22-tableau-extension-sfdc-writeback
HOT Session Materials for TC22
Written by Takashi Binns


# Overview
People often analyze CRM data within Tableau, to easily generate visualizations and gain insights in their data.  But once you identify a subset of data that you want to work with, how can you take action on that?  By using a Tableau dashboard extension, you can simply click a button and send that dataset to Salesforce for the next part of its journey.

By the end of this solution, a Tableau user should be able to interact with their dashboard to get a specific subset of data (filter, drill, etc) and then click a button to write that dataset to Salesforce.  The included workbook contains mock data for new leads, so it makes sense to write back to the Lead object in Salesforce.  An analyst might get a large list of potential leads and want to use Tableau to narrow down this list and only add leads that meet some specific criteria.

# Prerequisites
Tableau Desktop - We will need Tableau Desktop installed in order to demonstrate the integration.
Heroku Account – A free trial of Heroku.  Sign up for an account
Salesforce Developer edition Org – A free trial of Salesforce. Sign up for a Developer Edition Org. 
Github Account - A free account in Github.  Sign up for an account.
Workshop Files - Download and unzip the files from this link.

# Solution
The solution to this mini-hack is to leverage a dashboard extension to take data from the tableau workbook and push it to Salesforce. To do this, you will need to configure salesforce, deploy the dashboard extension, and configure the dashboard extension within your Tableau dashboard.

![https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic1.png?raw=true]

This dashboard extension is an open source project that can be configured to write data to any Salesforce object.  Once it’s been deployed, it will need to be configured so that it knows what Salesforce Object to write data to and what fields are required to do so.

## Deploy the dashboard extension web app
The dashboard extension is really just a web app that gets embedded into your Tableau workbook.  There are lots of ways to run web apps these days, but one of the easiest ways is to leverage Heroku.  If you have the web app code in your github repository, we can seamlessly deploy it straight to heroku without writing any code.

1. Login to your Github account, and search for the takashibinns/tableau-extension-salesforce-writeback repo.  Click on the Fork button to create your own copy of this repository.  

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic2.png?raw=true)

2.  Login to your Heroku Account and create a new app.  

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic3.png?raw=true)

You’ll need to give it a name, so pick something unique to yourself.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic4.png?raw=true)

The next step is to connect this Heroku app to your Github repository.  Set the deployment method to Github and then click the button to connect to Github.  You will get prompted to login to your Github account.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic6.png?raw=true)

Use the search box to find your forked repo, and click Connect.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic7.png?raw=true)

You will be prompted to specify which branch to deploy from, just select Master and then click on the Deploy Branch button.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic8.png?raw=true)

This process will take a few minutes, but you can see the log output showing its progress.  If all goes well, you should see a message stating that the Deploy to Heroku step was successful.  

If you click the View button it will open the web app in another browser tab.  This will display just a small blue Save button, and you can note the URL for accessing your dashboard extension.  We’ll need this URL later, when editing our .trex file.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic9.png?raw=true)

## Create a Connected App in Salesforce
In order for our dashboard extension to communicate with Salesforce, we need a way for it to authenticate.  In the Salesforce world, that means creating a Connected App.  Think of this as a way to map Salesforce Users to an external application (our dashboard extension) and specify exactly what permissions this external application requires.

1. Login to your Salesforce trial org and switch to the Setup tab.  Use the search bar to search for App Manager and click on the button for New Connected App.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic10.png?raw=true)

Give your connected app a name, contact email, and check the box to Enable OAuth Settings.  This will show more options, so enter a callback url (https://www.salesforce.com) and add the Manage User Data via APIs scope.  After clicking save the new connected app should be ready in ~10 minutes.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic11.png?raw=true)

2. Now you have a connected app, but no users are actually allowed to use it.  We want to configure it, so that a salesforce user can authenticate from our dashboard extension and leverage this connected app.

First note the Consumer Key and Consumer Secret.  Copy these values, as we’ll need them later.

For now click on the Manage button.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic12.png?raw=true)

Click on the Edit button and change the OAuth Policies -> Permitted Users setting to Admin approved users are pre-authorized.  You will also need to set the IP Restriction setting to Relax IP Restrictions.  Click the save button to get to the previous page.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic13.png?raw=true)

Now we can add Salesforce profiles that are allowed to use this connected app.  Click Manage Profiles and check the boxes next to each profile you want to allow.  In this mini-hack we can just use the System Administrator, but in a production environment you’d likely use more limited profiles.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic14.png?raw=true)

## Configure the dashboard extension

1. In heroku, navigate to your app’s Settings tab.  This tab has a section for adding Config Vars (environment variables) for the web app.  Add the following config vars, based on your Salesforce Connected App


CONSUMERKEY - Copy and paste from the Connected App’s manage connected apps page
CONSUMERSECRET - Copy and paste from the Connected App’s manage connected apps page
PROFILES - A comma separated list of profile names.  This guide just used System Administrator so you can enter that.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic15.png?raw=true)

Now we need to re-deploy the application to ensure those Config Vars get picked up.  Go back to the Deploy tab and click the Deploy Branch button again.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic16.png?raw=true)

2. One of the files downloaded as a prerequisite is named sf-writeback-extension.trex.

Open this file in a text editor, and view its XML.  Find the attribute named '<url>', and replace its value with the URL to your Heroku App.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic17.png?raw=true)

Save the file, and it’s ready to use in Tableau Desktop.

## Add the dashboard extension to your dashboard

1. Open the included tableau dashboard (LeadAnalysis.twbx) and make sure you’re on the Leads dashboard.  This is a standard Tableau dashboard that contains data about potential leads that we may want to upload to Salesforce.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic18.png?raw=true)

2. Drag a new Dashboard Extension object to the dashboard, and click the button for Access Local Extension.  Navigate to your .trex file and select it, in order to add it to your dashboard.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic19.png?raw=true)

You will see a popup asking to confirm the permissions of the dashboard extension, click the OK button.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic20.png?raw=true)
  
3. You should see the blue Save button appear in the dashboard extension.  Next, we need to configure the extension by first authenticating to Salesforce.  Click on the dashboard extension object in your dashboard so that you see the gray border around it and the toolbar.  Click on the toolbar’s carrot icon to view the menu and select Configure.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic21.png?raw=true)

You should see a prompt to log into Salesforce, so enter your trial org’s username and password then click Login.

After authenticating to Salesforce, you will see 2 dropdown menus.  The first is asking where to find your Tableau data.  Select the Lead Details sheet from the dropdown list (this contains all sheets within your dashboard, and this specific sheet is a hidden table with the lead data).  The second dropdown is asking what Salesforce object to write to.  Select Lead from the list of objects and then go to the Mapping tab.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic22.png?raw=true)
  
The Mapping tab lists all the fields available in the Salesforce object, and gives you an option to map it to a field from your Tableau sheet.  The screenshot below shows how the fields in our dataset map to the Lead object’s fields.  You don’t necessarily need to map every field, but objects often have required fields which need to be mapped.  When you’ve made these selections, click the Save button to finish configuration.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic23.png?raw=true)

## Verify it works

1. The easiest way to verify this is working, is to try it out! Either click the dashboard extension’s Save button right away to write all 5000 records to Salesforce or filter the dashboard first by interacting with the visualizations below.  Either way, you should see a green toast notification telling you how many records were written to the object.

!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic24.png?raw=true)
  

2. Maybe you don’t believe the dashboard extension’s notification? Well if you navigate to Salesforce’s Sales Console app in your browser and find the Lead object.  Make sure your view says All Open Leads and you will see all the leads newly added from Tableau.  This should have a Created Date equal to the current time (in PST) and the Owner field will be the Username you specified when authenticating in the dashboard extension.
 
!(https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic25.png?raw=true)
