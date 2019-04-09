# Developing an AppStudio for ArcGIS Application with Multiple Developers

## Intro
Now that developers have adopted [AppStudio for ArcGIS](https://doc.arcgis.com/en/appstudio/), they are beginning to work on increasingly complex projects, often requiring multiple developers.  As the scope increases from one developer to many, the question often arises: 
> “What is the best way to work in AppStudio with more than one developer?”

This is a question that has come up lately in Esri Professional Services, and one that my team and I grappled with for some time in the early stages of a multiple-developer AppStudio project.  Detailed below are some of the best practices my team and I have found to work for us, based on a combination of: basic Git and version control principles; pre-existing AppStudio workflows for single developers; as well as personal team preferences.

The workflows below rely on a few assumptions.  The first assumption is that Git or another version control system (VCS) is being used by all developers, and all developers are familiar with the basics of your chosen VCS.  For this article I will be using Git as the preferred VCS, so feel free to read “Git” as “insert your preferred version control system here”.  As in all development workflows, **Git is your friend**.  The second assumption is that you are already somewhat familiar with a single developer workflow working in AppStudio.


## Basic Dev Setup
The basic idea is that each developer effectively works in their own “silo”, in which they set up their own local git repositories, local AppStudio applications, and their own items in their ArcGIS Online organization.  The only shared resource in the development stage should be a git repository.

>> Insert dev setup image here

Each developer signs into AppStudio using their developer-licensed account and creates a new unregistered app for dev and testing purposes only.  The project code is copied into the local AppStudio app directory each time the application needs to be built in the could for testing.  There will exist only one registered “master” application in the main ArcGIS Online Organization, which has a unique app id.  Each developer’s unregistered local app will use the master app’s app id as the client_id parameter in the project’s appinfo.json file.  Every time the app runs and is authenticated against the main organization, it will give the master client_id and ArcGIS online will treat it as the “master” application.  

>> Insert app registration image here 

The setup steps are as follows:
#### Create a registered app one time in ArcGIS Online or Enterprise:
1. Add item > an Application > Type: application > enter title / tags > Add Item
2. Go to item and click settings, scroll down to application and click “Registered Info” 
3. Copy app id into the appinfo.json client_id parameter.  Make sure the updated appinfo.json is checked into version control.
4. Mark as authoritative if desired

#### Developer first time setup:
1.	Clone project into local git repository
2.	Sign into AppStudio with the developer account that is licensed for AppStudio development.  This can be:

    a.	a developer account created through the [developer website](https://developers.arcgis.com/)
    
    b.	or a developer license purchased through your organization.
    
    c.	See [here](https://developers.arcgis.com/pricing/compare-plans/) for licensing options.
3.	Create blank AppStudio Project but do not register it:

    a.	“New App”
    
    b.	“Starter” tab
   
    c.	“App”
    
    d.	Give it a title if desired: (username appname dev), i.e “cmosnick MyApp dev only”
    
    e.	Click “Create”

#### Each time before uploading and creating a cloud make in AppStudio:
Now that you have the project downloaded from source control and a new AppStudio application locally, you’ll need to connect the two, so that the local AppStudio app is always up to date with what is in source control.  I prefer to develop in the source-controlled git directory, instead of in the AppStudio local application directory, so that all my work stays in source control throughout the whole process.  I simply open the .qmlproject file through QtCreator and develop and run the code from there.

Because all the project code lives in the local git directory, the AppStudio application directory needs those files when the application is uploaded to ArcGIS Online for building in the cloud.  We need to set up some way to sync from the local git repository to the local AppStudio application directory.  You can choose to perform a copy and paste every time, or I chose to set up symbolic links so that the directories are always linked. 

1.	Copy / paste all files from local git repository to local AppStudio app location, **except** `iteminfo.json`:
    
    a.	In AppStudio, click “Files” to go to AppStudio app location

    b.	Copy and paste from local git repo to local AppStudio app directory:

        i.	All files **except** `iteminfo.json`
2.	Run, upload, and cloud make in AppStudio

>> Insert detailed developer view here

As you can see, all of the project files from the git repository are copied into the local AppStudio app’s location, except the iteminfo.json.  This is because the iteminfo.json file is specific to that local AppStudio item.  Most app-specific information and settings will be located in appinfo.json, which should be shared via version control and should be the same across all developers’ applications (so that they all have the same app settings and client_id).

This copying and pasting process can get tedious, especially if you make frequent cloud builds.  I automated this by creating symbolic links from my local git repo to the local AppStudio app directory.  Some possible solutions for automation include:

1.	creating symbolic links from the repo to the AppStudio directory. This works like *magic*
2.	writing a script to copy and paste from the git repo to the AppStudio directory
3.	running a grunt task to copy and paste upon git repo file changes


## Team Workflows
Now that all developers are set up, there are a few possible configurations to consider for deployment.  Configurations 1 & 2 highlight the fact that it does not matter whether developers are licensed within the same organization, and consequently where their dev versions of the app live.  This is because each developer’s workflow is “siloed”, and not dependent on the organization to which they belong.  For deployment, both configurations require the final build to be built by any user and copied to a designated location for distribution. 

>> Insert config 1 image here
>> Insert config 2 image here

For both configurations 1 & 2:
1.	The developer performs the development setup steps above
2.	The developer will upload the app to their org and perform a cloud make whenever a dev version is needed for a device.
3.	When a master app build is needed, one developer will have to perform a build and put it in the designated final build location

This build process has some potential drawbacks:
*	Confusion about which is the most up to date final version of the app
*	Confusion about the location of the final build
*	Inability to have a “final” item in ArcGIS online that any user can build to, and which can be distributed through AppStudio player

Configuration 3 solves these issues by adding one more “master” account which exists solely to curate the “master” app build item in the organization.  This “master” account owns a separate developer license through your organization.  Its credentials can optionally be shared among developers so that anyone can build to the final “master” application when needed.  As we saw before, the location of a developer’s licensing is negligible once the app is built and run with the master’s client_id.

>> Insert config 3 image here

When a master build is needed, one developer will:
1.	Log out of AppStudio if signed into personal dev account
2.	Log into AppStudio using the master user’s credentials
3.	Locate the master app in the organization and download to local AppStudio if needed
4.	Copy the most recent files from their git repo to the local AppStudio master app location
5.	Upload and make in AppStudio

With configuration 3, one final and “master” app exists in the ArcGIS organization, and serves as the authoritative build.  This item can be distributed via AppStudio Player as well as third party distributors. Consequently, an extra licensed developer account is needed.

## Summary:
|          | configuration 1 | configuration 2 | configuration 3 |
|----------|-----------------|-----------------|-----------------|
|Developers can be licensed in different organizations or same|**yes**|**yes**|**yes**|
|One registered “master” item in ArcGIS Organization with unique client_id (called App Id in ArcGIS Online)|**yes**|**yes**|**yes**|
|One final “Master” app in ArcGIS Organization|no|no|**yes**|
|Can be distributed by third party app distributors|**yes**|**yes**|**yes**|
|Can be distributed with app studio player|no|no|**yes**|
|Extra user / license needed|**no**|**no**|yes|

All scenarios have one “master” item in ArcGIS Online which is registered with an App Id for licensing and authentication purposes.  
