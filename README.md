# TC22-tableau-extension-sfdc-writeback
HOT Session Materials for TC22

Written by Takashi Binns
Edited for TC

# Overview
People often analyze CRM data within Tableau, to easily generate visualizations and gain insights in their data.  But once you identify a subset of data that you want to work with, how can you take action on that?  By using a Tableau dashboard extension, you can simply click a button and send that dataset to Salesforce for the next part of its journey.

By the end of this solution, a Tableau user should be able to interact with their dashboard to get a specific subset of data (filter, drill, etc) and then click a button to write that dataset to Salesforce.  The included workbook contains mock data for new leads, so it makes sense to write back to the Lead object in Salesforce.  An analyst might get a large list of potential leads and want to use Tableau to narrow down this list and only add leads that meet some specific criteria.

## Prerequisites

**Tableau Desktop** - We will need Tableau Desktop installed in order to demonstrate the integration. This has already been done for you.

**Salesforce Developer edition Org** – A free trial of Salesforce. [Sign up for a Developer Edition Org](https://developer.salesforce.com/signup) or use your own existing developer org. Note that the username does not have to be your real email. Copy your username down somewhere. You will also have to wait to get an email 

**Workshop Files** - Download and unzip the files from this [link](https://tableau.egnyte.com/dl/9KEl5FIRN0/Mini_Hack_Files.zip_).

*Note: If you need to copy-paste text into the VM at any time, use the lightning icon at the top left corner and select 'Type Text' -> 'Type Clipboard Text'.*

![clipboard](https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic28.png?raw=true)

## Solution

The solution to this mini-hack is to leverage a dashboard extension to take data from the tableau workbook and push it to Salesforce. To do this, you will need to configure salesforce, deploy the dashboard extension, and configure the dashboard extension within your Tableau dashboard.

![diagram](https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic1.png?raw=true)

This dashboard extension is an open source project that can be configured to write data to any Salesforce object.  Once it’s been deployed, it will need to be configured so that it knows what Salesforce Object to write data to and what fields are required to do so.

===

## Create a Connected App in Salesforce
In order for our dashboard extension to communicate with Salesforce, we need a way for it to authenticate.  In the Salesforce world, that means creating a Connected App.  Think of this as a way to map Salesforce Users to an external application (our dashboard extension) and specify exactly what permissions this external application requires.

1. Login to your Salesforce trial org and switch to the Setup tab.  Use the search bar to search for App Manager and click on the button for **New Connected App**.

![salesforce_app_manager](https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic10.png?raw=true)

Give your connected app the name 'minihack', contact email (this doesn't have to be real, so we can use the username you created), and check the box to **Enable OAuth Settings**.  This will show more options, so enter a callback url (https://www.salesforce.com) and under 'Available OAuth Scopes', add the **Manage User Data via APIs** scope.  Click 'Save and Continue'. After clicking save the new connected app should be ready in ~10 minutes.

![app_manager_setup](https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic11.png?raw=true)

2. Now you have a connected app, but no users are actually allowed to use it.  We want to configure it, so that a salesforce user can authenticate from our dashboard extension and leverage this connected app.

First note the **Consumer Key** and **Consumer Secret**.  Copy these values, as we’ll need them later.

For now click on the **Manage** button.

![manage_connected_apps](https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic12.png?raw=true)

Click on the **Edit Policies** button and change the OAuth Policies -> Permitted Users setting to **Admin approved users are pre-authorized**.  You will also need to set the IP Restriction setting to **Relax IP Restrictions**.  Click the save button to get to the previous page.

![dashboard_extension_setup](https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic13.png?raw=true)

Now we can add Salesforce profiles that are allowed to use this connected app.  Click **Manage Profiles** and check the boxes next to each profile you want to allow.  In this mini-hack we can just use the System Administrator, but in a production environment you’d likely use more limited profiles. After checking the 'System Administrator' option, click save.

![profiles](https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic14.png?raw=true)

===

## Configure the dashboard extension

The dashboard extension is really just a web app that gets embedded into your Tableau workbook.  There are lots of ways to run web apps these days, but for this mini-hack we can just run the web app locally.

1. Open a terminal window by clicking the windows button and typing cmd. Once it opens, clone the web app from Github by typing:

`git clone https://github.com/takashibinns/tableau-extension-salesforce-writeback.git`

2. Once the files are done downloading, navigate to 'Local Disk C:' -> 'Users' -> 'LabUser' -> tableau-extension-salesforce-writeback. Create a new file named '.env' in the folder by right clicking anywhere in the whitespace and select 'New' -> 'Text Cocument' and copy your consumer key and secret to the following environment variables. For the Profiles, you can customize this further if you were to create an API-specific profile, but for today, we can just use 'System Administrator'.

`PORT=8080`

`CONSUMERKEY=<consumer-key-from-connected-app>`

`CONSUMERSECRET=<consumer-secret-from-connected-app>`

`PROFILES="<Salesforce Profile>"`

**Important:** Save your file as 'all files' NOT a text file. It will show up as a file type ENV if you do this correctly. 

![file_named_env](https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic29.png?raw=true)


3. From your terminal, navigate to the 'tableau-extension-salesforce-writeback folder' run the following command to start up the dashboard extension web app

`cd tableau-extension-salesforce-writeback`

Hit enter.

`npm install`

Hit Enter. This will take a few seconds to load. 

`npm run dev`

Hit Enter.

You should now see the dashboard extension running in your web browser on port 3000.  It’s just a blue button. Once you see the save button, you can close the window. 

![localhost_extension](https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/Screen%20Shot%202022-05-06%20at%209.53.58%20AM.png?raw=true)

===

## Add the dashboard extension to your dashboard

1. Open the included tableau dashboard (LeadAnalysis.twbx) and make sure you’re on the Leads dashboard.  This is a standard Tableau dashboard that contains data about potential leads that we may want to upload to Salesforce.

![tab_dashboard](https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic18.png?raw=true)

2. Find the dashboard extension button in the bottom left corner of your dashboard layout menu. Drag a new Dashboard Extension object to the top right of the dashboard, and click the button for **Access Local Extension**.  Navigate to your .trex file and select it, in order to add it to your dashboard.

![add_extension_to_dash](https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic19.png?raw=true)

You will see a popup asking to confirm the permissions of the dashboard extension, click the OK button.

![allow_extension](https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic20.png?raw=true)
  
3. You should see the blue Save button appear in the dashboard extension.  Next, we need to configure the extension by first authenticating to Salesforce.  Click on the dashboard extension object in your dashboard so that you see the gray border around it and the toolbar.  Click on the toolbar’s carrot icon to view the menu and select **Configure**.

![configure](https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic21.png?raw=true)

You should see a prompt to log into Salesforce, so enter your trial org’s username (NOT your real email) and password then click **Login**.

After authenticating to Salesforce, you will see 2 dropdown menus.  The first is asking where to find your Tableau data.  Select the **Lead Details** sheet from the dropdown list (this contains all sheets within your dashboard, and this specific sheet is a hidden table with the lead data).  The second dropdown is asking what Salesforce object to write to.  Select **Lead** from the list of objects and then go to the **Mapping** tab.

![configure_2](https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic22.png?raw=true)
  
The Mapping tab lists all the fields available in the Salesforce object, and gives you an option to map it to a field from your Tableau sheet.  The screenshot below shows how the fields in our dataset map to the Lead object’s fields.  You don’t necessarily need to map every field, but objects often have required fields which need to be mapped.  When you’ve made these selections, click the **Save** button to finish configuration.

![configure_attributes](https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic23.png?raw=true)

===

## Verify it works

1. The easiest way to verify this is working, is to try it out! Either click the dashboard extension’s **Save** button right away to write all 5000 records to Salesforce or filter the dashboard first by interacting with the visualizations below.  Either way, you should see a green toast notification telling you how many records were written to the object.

![test_dashboard](https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic24.png?raw=true)
  

2. Maybe you don’t believe the dashboard extension’s notification? Well if you navigate to Salesforce’s **Sales Console app** in your browser and find the Lead object.  Make sure your view says 'All Open Leads' and you will see all the leads newly added from Tableau.  This should have a Created Date equal to the current time (in PST) and the Owner field will be the Username you specified when authenticating in the dashboard extension.
 
![see_leads](https://github.com/madelinefromtableau/TC22-tableau-extension-sfdc-writeback/blob/main/pic25.png?raw=true)
