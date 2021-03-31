# Node.js authentication proxy used for oAuth communication with IBM Cloud AppID

The code in this repository can be used to setup an authentication proxy between a Frontend application such as VueJS and a IBM Cloud AppID.

First create a new "App ID" service instance in IBM Cloud.

Open the newly created Service instance:
- Add a new application
- Give it a name.
- Use "regularwebapp" as application type

When you click on the newly created app you should see a JSON structure with all info such as : clientId, tenantId and oAuthServerURL.
Take note as you will need these values in the next step !


# Setup in a local environment

* Clone this repo to a local folder on your machine
* Create a file ".env" - see sample below

```
APPID_OAUTHSERVERURL=https://us-south.appid.cloud.ibm.com/oauth/v4/8f1d603d-2a03-4fa5-8d3b-2218c200929e
APPID_CLIENTID=xxx
APPID_SECRET=xxx
REDIRECT_URL_CALLBACK=http://localhost:3000/callback
REDIRECT_URL_WEB_APP=http://localhost:8080/loginwithtoken
```
* Replace the values with the values obtained previously
* Install the required node modules @ start the application

```
# npm install
# npm run start
```
* Quick application test : Open a browser on <http://localhost:3000>
* You should now see the AppID login page.

	You will probably get an error "Invalid redirect_uri".
To solve this, goto to your AppID service instance -> Manage Authentication -> Authentication Settings and add <http://localhost:3000/*> as a web redirect URL.


* After successful login you will see a redirect URL in your browser which containes 3 query parameters:
	* name
	* email
	* id_token
	* access_token
* Now it's up to your Frontend application to handle these query params...

# Deploy Node.js authentication to RedHat OpenShift

You can either use your own repo in github as a clone of this one or start from this repo directly.

Open up your RedHat Openshift Web-console: Goto the Developer perspective and click "Add" from the menu on the left.

- Select Add from Dockerfile
- Fill in the "Git Repo URL" which points to your project on github or this repo
- Click "Show Advanced Git Options" and specify the "Git Reference" to point to your branch. This is only needed if your branch is different than "master" !
- Change the 'Container Port' to 3000
- Change the 'Application Name' and 'Name' accordingly or accept the defaults
- Finally Click "Create"

This should result in a full deployment of the application.
the "Build Config" will create a "Build" to construct the final image.
Once the Deployment has completed, you can verify the application is working correctly by clicking on the generated route URL.

**Modify the environment variables for your application

* Open the "Build Config" in the OpenSshift Web-console
* Click on the "Environment" tab
* Add the following variables and fill in the correct values to reflect your AppID config and application URL's (either local or deployed URL's:
	* APPID_OAUTHSERVERURL=https://eu-gb.appid.cloud.ibm.com/oauth/v4/70ef0d78-2967-4444-7777-85939e5d4ca6
	* APPID_CLIENTID=xxxxx
	* APPID_SECRET=xxxx
	* REDIRECT_URL_CALLBACK=http://localhost:3000/callback
	* REDIRECT_URL_WEB_APP=http://localhost:8080/loginwithtoken

**Adding a Github Webhook to automatically update the Openshift deployment as soon as new code is pushed into the repository:**

- Goto the "Build Config" and select your specific Build Config
- Click the Github "Copy URL with Secret"
- Goto to your Github project on "github.com" and select "Settings" -> Webhooks
	- Add a new webhook and paste the URL copied in previous step in "Payload URL"
	- Select "application/json" as Content Type
	- Click "Add webhook"
- The result of this configuration will trigger a new build run on Openshift as soon as new code gets committed into the repository.

