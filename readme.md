## Server Side Resources
---
[Go Factory](http://www.go-factory.net) provides group creation, management, collaboration, and communication services for the mobile enterprise through a native SDK and enterprise-grade, cloud-based back end. In this document, we explore the potential of next-generation, custom, mobile solutions based on real-time collaboration and communication, powered by Go Factory’s Go Machine™.

The Go Machine for GroupsSDK is a fully hosted solution powered by the Go Machine, a highly scalable and fault tolerant telco grade backend. To check out the SDK in action, we created a consumer application to showcase everything you can do with our GroupsSDK called [Go-Matic](http://www.go-matic.com "Go-Matic").

The backend Go Machine platform not only hosts all the group functionality through RPC calls, it supports some unique features not found in other solutions that can all be optionally customized.  

* Customizable landing page for all sharing (i.e. photos, audio, video, etc)
* A seamless join/install by invitation feature with customizable landing page
* And a customizable collaborative publisher.

If you prefer not to want to deal with this, fear not, everything works out of the box, just with a default look and feel.

Using the go machine server resource template for your Groups enabled app is fairly straight forward and easy to set up while offering a good deal of control.  When you create your Go Factory account for your application, you will have the option to either deploy a non modifiable generic look and feel, or you will be asked to provide your remote SCM repository to the your resource template that you control.

**NOTE:** If you do not specify your git repo, a template with a default look and feel will be deployed that is not modifiable.  If at some point later you decide you want to customize the various landing pages fear not, simply update your account with a git repository and it will used instead. In this case, simply create your Go Factory account and you are good to go.

To set up your Go Machine resources directory is pretty easy, and requires three steps to get started:

1. First, create a new Github repository called gomachine-resources. Be sure check "Initialize this repository with a README". 
2. Add gomachine as a collaborators. If your account is set up for teams, only give pull permissions. 
3. You will need to create a duplicate of a gomachine-template repo without forking as forking a public repo makes your repo public and I dont think you will want that.  See [Duplicating a repo](https://help.github.com/articles/duplicating-a-repo).

In your terminal:
    
    cd <working or temporary directory>
    # Its better to do this outside of your root source directory

    git clone --bare https://github.com/gomachine/gomachine-template.git
    # Make a bare clone of the repo
    
    cd gomachine-template.git
    $ git push --mirror https://github.com/<myaccount>/gomachine-resources.git
    # Mirror-push to the new repo
    
    cd ..
    $ rm -rf gomachine-template.git
    # Remove our temporary local repo
    
    cd <development directory>
    $ git clone https://github.com/<myaccount>/gomachine-resources.git

**WARNING**
If you try to mirror push the bare cloned repo before you created and initialized one in your account, you will get the following error message. The repo must be initialized.

    $ git push --mirror https://github.com/myaccount/gomachine-resources.git
    fatal: https://github.com/myaccount/gomachine-resources.git/info/refs not found: did you run git update-server-info on the server?

Thats it. 

When greating your account, set your SCM to git and your SCM Url to <u>git@github.com:&lt;myaccount&gt;/gomachine-resources.git</u> where &lt;myaccount&gt; is your github account.

The template repo comes with everything for a default look and can be used as is.  Customizing the look is done primarily through changes to the CSS files (iphone.css, android.css, and other.css), and replacing the image assets with your own.  

At this time, the go machine will only pull the master branch and only when the developer does so from the management console.  At some point in the future we may support multiple branches for the dev sandbox and production.  The one exception is the initial pull when the account is set up. 

## Requesting Your Go Machine Resources
The Go Machine is not meant to be a general purpose web server and thus will return a 404 on any index request.  To request resources, for example the background in the CSS, prepend your APIKey to the resource url.

    body {
        background-color:#e4aa58;
        background-image:url('/34598udf23/images/my_background.png');
        background-repeat:no-repeat;
        background-size: 100%;
        margin:25px 25px;

        font-family:"Verdana";
        font-size:36px;
        color:#261600;
    }

Where my_background.png would be an asset in your www/images subdirectory.  

**NOTES:** 

* The Go Machine web process enforces a strict expression matching and URL rewritting.
* Be sure to keep the file names intact, otherwise you will have a broken site and everyone will be sad.
* Resources are restricted to standard media, HTML, JS, and CSS.
* Any index.html request will be ignored.

## Assets You Should Replace

The following are file assets that you should replace with your own.

File         		| What its for
-------------		|-------------
background.png      | A background image
logo_medium.png 	| A 200x200 PNG of your logo 
logo_large.png		| A 300x300 PNG of your logo 
missing_cover.png	| A 400x400 PNG to use if their is no image available for the Publisher 
missing_member.png | A 400x400 PNG to use when a member ID is requested but was not part of the group
missing_profile.png | A 100x100 to user if the member is missing a profile image
system.png | A 100x100 profile image of the system (used in system chat messages)
button_install.png | A 573x142 PNG to use for the Install Button on the invitation
button_join.png | A 573x142 PNG to use for the Join Button on the invitation
button_lets_go.png | A 573x142 PNG to use for the Let's Go Button on after the new install

## Directory Structure

The Go Machine expects the following directory structure:

    - templates
    |	- share_landing.dtl
    |
    - www
    	- css
    	|	|
    	|	- iphone.css
    	|	- android.css
    	|	- other.css
    	|
    	- images

You are free to add necessary resources to support your share_landing template.

## Customizing the Sharing Landing Page
The shared landing page is implemented using a Django template in the templates sub-directory.  The server passes in the following bound variables:

    {image_source, ImageSource} - A fully qualified http URL to use in an <img src="…"/> tag
    {media_source, MediaSource} - A fully qualified http URL to the media
    {content_type, ContentType} - The content-type of the image or media
    {apikey, APIKey} - Your APIKey necessary to prefix any href, links, or scripts in your repo

You are free to modify this file as you see fit, just be sure that the image or media is handled.  This template is rendered whenever a shared URL is clicked on, either from an email, a consumer site like Facebook or Twitter, or from an internal intranet.

## Supporting Join by Invitation

One of the major challenges of any mobile application with collaborative and group requirements is dealing with the situation where a user wishes to invite a friend or colleague to join them in their group in your app.  What happens if their friend or colleague doesn't have your application installed?

The Groups SDK includes functionality to support this very situation - allowing your app to spread through invitations if you choose.  Sending invitation is easy with the Groups SDK.  Simply create the invitation URL by calling the inviteToChannel method in the Groups SDK.  This method returns an URL which your user is free to share either in a text, email, or posting it on sites like Facebook depending upon the nature of your app.  In some cases you may want to restrict joining by invation to email.

Our solution is able to determine if your application is installed on the recipients device.  If so, your application is launched and the recipient joins the group already in progress.  If not, we send the recipient to the appropriate app store or play store so they can download and install your app.  Once downloaded, the user joins the group in progress.

For enterprise applications, this method can still be used, except instead of sending the new install to the appropriate consumer app store, the Go Machine would instead redirect to the internal mobile device management (MDM) solution. (NOTE: Support for MDM solutions is not yet available).

To customize the look and feel of the server side pages of the invitation process, simply modify the CSS files for each device which includes changing the background, logo and look and feel of the three buttons.

The invitation process is complete and seamless flow from the users perspective, but will require you to implement three steps.  When your user clicks on their invitation on their device, the pages served to the devices browser attempts to launch the app using the customer app URI that you provided when setting up your account.  If you have not done so, please go to the management console and do this now.  If the app fails to launch, the page the redirects the user to the appropriate app store for their device so they can download and install your app.

Once the user has installed your app and they launch it for the first time, you will need to implement the following logic:

* On application launch, determine if it is the first time use and if so, have your user configure their profile, including the minimal information needed for the groups service (user name and profile image).
  
* Next, you will need to prompt the user if they they installed the app from an application.  If they answer yes, then call startSessionInstalledByInvitation to start the session.  The completion block or anonymous block will be performed where you will asked to send over the profile information.  

* At this, your handleServerRequest callback will also be called with an URL request back to the backend that you will need to open.  This will result in the switching back to the server page which will then pick up the details such as the channel reference and who invited the user, and then launch your app again which will result in the user joining the group they were invited to, already in progress.

Sounds complicated, but it is fairly easy. See the GroupsSDK documentation for code examples.

## Supporting your Journal (aka Collaborative Publisher)

[TO BE ADDED]
   