# IBM Client Developer Advocacy App Modernization Series

## Lab - HTTPSession replication in WebSphere Liberty on OpenShift

## Overview

WebSphere Liberty has a feature called **sessionCache-1.0** which provides distributed in-memory HttpSession caching. The **sessionCache-1.0** feature builds on top of an existing technology called JCache (JSR 107), which offers a standardized distributed in-memory caching API. However, even though the feature builds on top of JCache, no direct usage of JCache API is necessary in your application, since Liberty handles the session caching in its HttpSession implementation. In fact, if your application is already using HttpSession caching, it can benefit from **sessionCache-1.0 without making any code changes.**

In this lab you'll use these  capabilities  to deploy and test  a small Java EE app on OpenShift. You'll use an Open Source JCache provider called [Infinispan](https://infinispan.org) to provide the implementation of the JCache support that is included in Liberty. Note that any compliant JSR 107 product can be used in this manner with WebSphere Liberty.

### Step 1: Logon into the OpenShift Web Console and to the OpenShift CLI

1.1 Login into the OpenShift web console using  your user credentials

1.2 From the OpenShift web console click on your username in the upper right and select **Copy Login Command**

   ![Copy Login Command](images/ss0.png)

1.3 Paste the login command in a terminal window and run it (Note: leave the web console browser tab open as you'll need it later on in the lab)

### Step 2: Create ImageStreams for the Open Liberty base image and Infinispan server image

2.1 Set an environment variable for your *studentid* based on your user identifier from the instructor (e.g. **user001**)

    ```bash
    export STUDENTID=userNNN
    ```
2.2 Create a new OpenShift project for this lab

   ```bash
   oc new-project srpl-$STUDENTID
   ```

2.3 Tag the Docker Hub Open Liberty  image to create a local ImageStream

   ```
   oc tag docker.io/open-liberty:latest open-liberty:latest
   ```

2.4 Tag the Docker Hub Infinispan server image to create a local ImageStream

   ```
   oc tag docker.io/infinispan/server:latest infinispan-server:latest
   ```

### Step 3: Install the sample  app and Infinispan server using templates  

3.1  From the client terminal window clone the following Git repo with the app used in this lab

   ```bash
   git clone https://github.com/IBMAppModernization/simple-http-session-app.git
   cd simple-http-session-app
   ```

3.2 Add the sample app  and Infinispan server templates to your project

   ```bash
   oc create -f openshift/templates/infinispan
   ```

3.3 In your Web console browser tab make sure you're in your new  project (top left) and click on **Add to Project -> Select from Project** (top right)

   ![View All](images/ss1.png)

3.4 Click on the **Infinispan server** template and then  Click **Next**.

3.5 Click **Next**, accept all the defaults and click **Create**

3.6 Click  **Continue to the project overview**

3.8 Wait until the Pod for the Infinispan server shows as running (and ready)

   ![Launch app](images/ss2.png)

3.9 In your Web console browser tab  click on **Add to Project -> Select from Project** (top right)

3.10 Click on the **Simple HttpSession sample on Liberty** template and then Click **Next**.

3.11 Click **Next**, accept all the defaults and click **Create**

3.12 Click  **Continue to the project overview**

3.13 Wait until the Pods for the sample  app shows as running (and ready) then click on the route to get to the app's endpoint

  ![Wait for app](images/ss3.png)

### Step 4: Test the sample app

When you bring up the app in a new browser session the banner on the web page will say  **Hello stranger**. A new HTTPSession object is created and it's state is replicated to all the pods. When the app encounters an existing HTTPSession object the banner message will change to **Welcome back friend**.  

4.1 When the app appears in your browser verify that the banner says  **Hello stranger**

   ![Running app](images/ss4.png)

4.2 Keep refreshing the URL and verify that the POD IP address changes each time and the current state of the  session data is maintained and used by all pods (i.e. the banner should say **Welcome back friend**  and the access count should keep incrementing). Keep this browser tab open as you'll need it for further testing.

   ![Session state shared](images/ss5.png)

4.3 Next you'll verify that if you increase the number of replicas the new replicas will automatically access the shared session state. In your Web console browser tab increase the number of pods  for the **simple-session** app to 3 as illustrated below:

   ![New pod](images/ss6.png)

4.4 Wait for the 3rd pod to be ready (ie circle around number of pods will be a  uniform  color)

   ![New pod ready](images/ss7.png)

4.5 Go back to the browser tab where you had the **simple-session** app running and keep refreshing the page. Verify that the 3rd pod is now being accessed and that the access count is never reset to 0.

## Cleanup

Run the following commands to cleanup (note: you can copy all the commands at once and post them into you command window)

   ```
   oc delete all,routes,is --selector app=simple-session
   oc delete all --selector app=infinispan-server
   oc delete is infinispan-server
   oc delete is open-liberty
   ```

## Summary
Congratulations. You used the **session-cache** feature in WebSphere Liberty along with the Open Source **Infinispan Data Grid** to demonstrate HttpSession replication in OpenShift. This allowed you to  deploy  a stateful application in a stateless manner (i.e each application pod can be deleted or replaced at anytime without losing application state).
