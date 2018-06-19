![](images/HeaderPic.png "Microsoft Cloud Workshops")

<div class="MCWHeader1">
Containers and DevOps
</div>

<div class="MCWHeader2">
Hands-on lab unguided
</div>

<div class="MCWHeader3">
March 2018
</div>

Information in this document, including URL and other Internet Web site references, is subject to change without notice. Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.

Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.

The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.
© 2018 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at https://www.microsoft.com/en-us/legal/intellectualproperty/Trademarks/Usage/General.aspx are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

**Contents**

<!-- TOC -->

- [Abstract and learning objectives](#abstract-and-learning-objectives)
- [Overview](#overview)
- [Solution architecture](#solution-architecture)
- [Requirements](#requirements)
- [Exercise 1: Create and run a Docker application](#exercise-1-create-and-run-a-docker-application)
    - [Task 1: Test the application](#task-1-test-the-application)
    - [Task 2: Enable browsing to the web site](#task-2-enable-browsing-to-the-web-site)
    - [Task 3: Create a Dockerfile](#task-3-create-a-dockerfile)
    - [Task 4: Create Docker images](#task-4-create-docker-images)
    - [Task 5: Run a containerized application](#task-5-run-a-containerized-application)
    - [Task 6: Setup environment variables](#task-6-setup-environment-variables)
    - [Task 7: Push images to Azure Container Registry](#task-7-push-images-to-azure-container-registry)
- [Exercise 2: Deploy the solution to Azure Container Service](#exercise-2-deploy-the-solution-to-azure-container-service)
    - [Task 1: Tunnel into the Azure Container Service cluster](#task-1-tunnel-into-the-azure-container-service-cluster)
    - [Task 2: Deploy a service using the Kubernetes management dashboard](#task-2-deploy-a-service-using-the-kubernetes-management-dashboard)
    - [Task 3: Deploy a service using Kubernetes REST API](#task-3-deploy-a-service-using-kubernetes-rest-api)
    - [Task 4: Explore service instance logs and resolve an issue](#task-4-explore-service-instance-logs-and-resolve-an-issue)
    - [Task 5: Test the application in a browser](#task-5-test-the-application-in-a-browser)
- [Exercise 3: Scale the application and test HA](#exercise-3-scale-the-application-and-test-ha)
    - [Task 1: Increase service instances from the Kubernetes dashboard](#task-1-increase-service-instances-from-the-kubernetes-dashboard)
    - [Task 2: Increase service instances beyond available resources](#task-2-increase-service-instances-beyond-available-resources)
    - [Task 3: Restart containers and test HA](#task-3-restart-containers-and-test-ha)
- [Exercise 4: Setup load balancing and service discovery](#exercise-4-setup-load-balancing-and-service-discovery)
    - [Task 1: Create a public load balancer for a service](#task-1-create-a-public-load-balancer-for-a-service)
    - [Task 2: Scale a service without port constraints](#task-2-scale-a-service-without-port-constraints)
    - [Task 3: Update a service to support dynamic service discovery without a load balancer](#task-3-update-a-service-to-support-dynamic-service-discovery-without-a-load-balancer)
    - [Task 4: Update an external service to support dynamic discovery with a load balancer](#task-4-update-an-external-service-to-support-dynamic-discovery-with-a-load-balancer)
    - [Task 5: Adjust CPU constraints to improve scale](#task-5-adjust-cpu-constraints-to-improve-scale)
    - [Task 6: Perform a rolling update](#task-6-perform-a-rolling-update)
- [After the hands-on lab](#after-the-hands-on-lab)

<!-- /TOC -->


## Abstract and learning objectives 

Build a PoC to deliver a multi-tenant web app hosting solution leveraging Azure Container Service (AKS), Docker containers, and Linux nodes.???

Attendees will be better able to deploy Docker-based applications and scale them with Azure Container Service and Kubernetes orchestration. In addition,

-   Create and run a Docker Application

-   Deploy to the Azure Container Service (AKS)

-   Implement load balancing and service discovery

-   Scale the application and test availability

## Overview

Fabrikam Medical Conferences (FabMedical) provides conference website services tailored to the medical community. They are refactoring their application code, based on node.js, so that it can run as a Docker application, and want to implement a POC that will help them get familiar with the development process, lifecycle of deployment, and critical aspects of the hosting environment. They will be deploying their applications to Azure Container Service (AKS) and want to learn how to deploy containers in a dynamically load-balanced manner, discover containers, and scale them on demand.

In this hands-on lab, you will assist with completing this POC with a subset of the application code base. You will create a build agent based on Linux, and an Azure Container Service cluster for running deployed applications. You will be helping them to complete the Docker setup for their application, test locally, push to an image repository, deploy to the cluster, and test load-balancing and scale.

**IMPORTANT: Most Azure resources require unique names. Throughout these steps you will see the word "SUFFIX" as part of resource names. You should replace this with your Microsoft email prefix to ensure the resource is uniquely named.**

## Solution architecture

Below is a diagram of the solution architecture you will build in this lab. Please study this carefully, so you understand the whole of the solution as you are working on the various components.

The solution will use Azure Kubernetes Service (AKS) which means that the container cluster topology is provisioned according to the number of requested nodes. The proposed containers deployed to the cluster are illustrated below, with MongoDB remaining as a managed service:

![The solution will use Azure Kubernetes Service (AKS). The proposed containers deployed to the cluster are illustrated in this diagram, with MongoDB remaining as a managed service.](images/Hands-onlabunguided-ContainersandDevOpsimages/media/image3.png)

Each tenant will have the following containers:

-   **Conference Web site**: the SPA application that will use configuration settings to handle custom styles for the tenant

-   **Admin Web site**: the SPA application that conference owners use to manage conference configuration details, manage attendee registrations, manage campaigns and communicate with attendees

-   **Registration service**: the API that handles all registration activities creating new conference registrations with the appropriate package selections and associated cost

-   **Email service**: the API that handles email notifications to conference attendees during registration, or when the conference owners choose to engage the attendees through their admin site

-   **Config service**: the API that handles conference configuration settings such as dates, locations, pricing tables, early bird specials, countdowns and related

-   **Content service**: the API that handles content for the conference such as speakers, sessions, workshops and sponsors

## Requirements

1.  Microsoft Azure subscription must be pay-as-you-go or MSDN.

    -   Trial subscriptions will *not* work.

    -   You must have rights to create a service principal as discussed in Task 9: Create a Service Principal --- and this typically requires a subscription owner to log in. You may have to ask another subscription owner to login to the portal and execute that step ahead of time if you do not have the rights.

    -   You must have enough cores available in your subscription to create the build agent and Azure Container Service cluster in Task 5: Create a build agent VM and Task 10: Create an Azure Container Service (AKS) cluster. You'll need eight cores if following the exact instructions in the lab, more if you choose additional agents or larger VM sizes. If you execute the steps required before the lab, you will be able to see if you need to request more cores in your sub.

2.  Local machine or a virtual machine configured with:

    -   A browser, preferably Chrome for consistency with the lab implementation tests

    -   Command prompt

        i.  On Windows, you will be using Bash on Ubuntu on Windows, hereon referred to as WSL.

        ii. On Mac, all instructions should be executed using bash in Terminal.

3.  You will be asked to install other tools throughout the exercises.

**VERY IMPORTANT: You should be typing all of the commands as they appear in the guide, except where explicitly stated in this document. Do not try to copy and paste from Word to your command windows or other documents where you are instructed to enter information shown in this document. There can be issues with Copy and Paste from Word that result in errors, execution of instructions, or creation of file content.**


## Exercise 1: Create and run a Docker application

**Duration**: 40 minutes

In this exercise, you will take the starter files and run the node.js application as a Docker application. You will create a Dockerfile, build Docker images and run containers to execute the application.

**NOTE: Complete these tasks from the WSL window with the build agent session.**

### Task 1: Test the application

The purpose of this task is to make sure you can run the application successfully before applying changes to run it as a Docker application.

1.  Run the API and web applications in that order, by navigating to their respective folders and running these commands:

    -   npm install

    -   nodejs ./server.js &

2.  Test the results by curling these URLs and observing the JSON responses:

    API Application:
    ```
    curl <http://localhost:3001/speakers>
    ```
    
    Web Application:
    ```
    curl <http://localhost:3000>
    ```

### Task 2: Enable browsing to the web site

In this task, you will open a port range on the agent VM so that you can browse to the application for testing.

1.  From the Azure portal, select the Network Security Group associated with the build agent VM. Create an inbound security rule to allow TCP calls over port range 3000-3010.

2.  Find the IP address for the VM and browse to that IP address with port 3000.

3.  You can browse to the website and see the speakers and session pages, you can reach the web application from a browser and have correctly opened the port on the agent VM.

4.  Once you have seen the website running, you can stop the running node processes with this command:

    ```
    killall nodejs
    ```
### Task 3: Create a Dockerfile

In this task, you will create a new Dockerfile that will be used to run the API as a containerized application.

**NOTE: You will be working in a Linux VM without friendly editor tools. You must follow the steps very carefully to work with Vim for a few editing exercises if you are not already familiar with Vim.**

1.  From the content-api folder, using Vim create a new file named Dockerfile.

2.  Create a Dockerfile based on the node:argon image.

3.  Include instructions to copy the source files for the application to the image and to run npm install.

4.  Expose port 3001 and put the startup command npm

5.  Verify the file contents, to ensure it saved as expected.

### Task 4: Create Docker images

In this task, you will create Docker images for the application -- one for the API and another for the web application. Each image will be created via Docker commands that rely on a Dockerfile.

1.  From the content-api directory, build a Docker image using the Dockerfile you created.

2.  From the content-web directory, build a Docker image with the existing Dockerfile in the folder.

3.  Once both images are successfully built, you will see three images now exist when you run the Docker images command. One for API, one for web, and another for node.

### Task 5: Run a containerized application

The web application container will be calling endpoints exposed by the API application container. In this exercise, you will create a bridge network so that containers can communicate with one another, and then launch the images you created as containers in that network.

1.  On the build agent VM create a new Docker network named "fabmedical".

2.  Start the API and web containers with these commands:

    ```
    docker run -d -p 3001:3001 --name api --net fabmedical content-api

    docker run -d -P --name web --net fabmedical content-web
    ```

3.  Test the API application by curling the speakers URL once again.

4.  Test the web application by curling the site but using the dynamic port assigned to the container from the run command.

### Task 6: Setup environment variables

In this task, you will configure the web container to communicate with the API container using an environment variable. You will modify the web application to read the URL from the environment variable, rebuild the Docker image, and then run the container again to test connectivity.

1.  From WSL, stop and remove the web container.

2.  Edit content-web\\data-access\\index.js and locate the TODO item and modify the code so that the contentApiUrl variable will be set to an environment variable.

3.  Edit the Dockerfile in content-web to add an environment variable CONTENT\_API\_URL matching http://localhost:3001.

4.  Rebuild the web Docker image.

5.  Create and start the image passing the correct URI to the API container as an environment variable, then check the port that the container is running on.

    ```
    docker run -d -P --name web --net fabmedical --e CONTENT_API_URL=http://api:3001 content-web
    ```
6.  Curl the speakers path again, using the port assigned to the web container.

7.  Stop and restart the container so that it uses port 3000 and then browse to the URL at the build agent IP address with that port to view the web application.

### Task 7: Push images to Azure Container Registry

To run containers in a remote environment, you will typically push images to a Docker registry, where you can store and distribute images. Each service will have a repository that can be pushed to and pulled from with Docker commands. Azure Container Registry (ACR) is a managed private Docker registry service based on Docker Registry v2.

In this task, you will push images to your ACR account, version images with tagging, and run containers from an image in your ACR.

1.  Gather credentials for your ACR in the Azure Portal.

2.  Login to your ACR from the build agent VM.

Tip: Make sure to specify the fully qualified registry login server (all lowercase).

3.  Tag your images to match your ACR account name. You may also specify the \"fabmedical\" namespace to avoid clutter in the root of the registry.

    ```
    docker tag content-web [LOGINSERVER]/fabmedical/content-web

    docker tag content-api [LOGINSERVER]/fabmedical/content-api
    ```
4.  Push the API and web images to the ACR.

5.  Assign a new tag "v1" to both images and push the newly tagged version of the images to the ACR.

6.  Verify the tagged images are created in the ACR.

7.  From your build agent VM, perform a pull on each image to validate pulls are working from the VM.

## Exercise 2: Deploy the solution to Azure Container Service

**Duration**: 30 minutes

In this exercise, you will connect to the Azure Container Service cluster you created before the hands-on lab, and deploy the Docker application to the cluster using Kubernetes.

### Task 1: Tunnel into the Azure Container Service cluster

In this task, you will gather the information you need about your Azure Container Service cluster to connect to the cluster and execute commands to connect to the Kubernetes management dashboard from your local machine.

1.  Verify that Azure CLI is working correctly and that you are logged in and connected to the correct default subscription.

2.  Verify that Kubernetes CLI is working correctly.

3.  Configure kubectl to connect to the Kubernetes cluster.

    ```
    az aks get-credentials --name fabmedical-SUFFIX --resource-group fabmedical-SUFFIX-2
    ```

4.  List the cluster nodes to verify the connection is working.

5.  Create an SSH tunnel linking a local port (8001) on your machine to port 80 on the management node of the cluster. Execute the command below replacing the values accordingly:
    
**NOTE: After you run this command, it may work at first and later lose its connection, so you may have to run this again to reestablish the connection. If the Kubernetes dashboard becomes unresponsive in the browser this is an indication to return here and check your tunnel or rerun the command.**

    ```
    az aks browse --name fabmedical-SUFFIX --resource-group fabmedical-SUFFIX-2
    ```

6.  Open a browser and navigate to the Kubernetes management dashboard.

### Task 2: Deploy a service using the Kubernetes management dashboard

In this task, you will deploy the API application to the Azure Container Service cluster using the Kubernetes dashboard.

1.  From the Kubernetes dashboard create a new deployment for the API application, based on the API image you pushed to your ACR. Use the following settings:

    -   Set Number of pods to 1

    -   Set Service to "Internal"

    -   Use 3001 for Port and 3001 for Target port

    -   Use 0.125 for the CPU requirement

    -   Use 128 for the Memory requirement

    a.  Once the deployment completes, edit the deployment, set replicas to 0 (so that you can redeploy by scaling) and add the following JSON snippet below the "name" property in the container spec.
    
    ```
    "ports": [
        {
            "containerPort": 3001,
            "hostPort": 3001
        }
    ],
    ```

    ![Screenshot showing the containerPort and hostPort updated to port 3001 for the container. ](images/Hands-onlabunguided-ContainersandDevOpsimages/media/image4.png)

2.  After changing the deployment settings, scale the service from 0 to 1.

3.  From the Workloads view, you can see the deployed objects and the replica sets that correspond to your two API deployments. After the deployment is successful, it will transition to a Running state.

### Task 3: Deploy a service using Kubernetes REST API

In this task, you will make HTTP requests using kubectl. These requests will deploy the web application using the REST API.

1.  Open a text editor such as Vim and copy the following text into that editor. Save the file with a name such as kubernetes-web.yaml.
    ```
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
    labels:
        app: web
    name: web
    spec:
    replicas: 1
    selector:
        matchLabels:
        app: web
    strategy:
        rollingUpdate:
        maxSurge: 1
        maxUnavailable: 1
        type: RollingUpdate
    template:
        metadata:
        labels:
            app: web
        name: web
        spec:
        containers:
        - image: [LOGINSERVER].azurecr.io/fabmedical/content-web
            env:
            - name: CONTENT_API_URL
            value: http://api:3001
            livenessProbe:
            httpGet:
                path: /
                port: 80
            initialDelaySeconds: 30
            periodSeconds: 20
            timeoutSeconds: 10
            failureThreshold: 3
            imagePullPolicy: Always
            name: web
            ports:
            - containerPort: 80
            hostPort: 80
            protocol: TCP
            resources:
            requests:
                cpu: 1000m
                memory: 128Mi
            securityContext:
            privileged: false
            terminationMessagePath: /dev/termination-log
            terminationMessagePolicy: File
        dnsPolicy: ClusterFirst
        restartPolicy: Always
        schedulerName: default-scheduler
        securityContext: {}
        terminationGracePeriodSeconds: 30
    ---
    apiVersion: v1
    kind: Service
    metadata:
    labels:
        app: web
    name: web
    spec:
    ports:
    - name: web-traffic
        port: 80
        protocol: TCP
        targetPort: 3000
    selector:
        app: web
    sessionAffinity: None
    type: LoadBalancer

    ```

2.  Edit this file and update the \[LOGINSERVER\] entry to match the name of your ACR login server.

3.  Deploy the application described by the YAML file using kubectl as follows:

    ```
    kubectl apply -f kubernetes-web.yaml
    ```

4.  From the Kubernetes dashboard, take a look at the state of the healthy API deployment and then take a look at the unhealthy "web" deployment. You will fix this in the next task.

### Task 4: Explore service instance logs and resolve an issue

In this task, you will determine why the "web" service deployment is unhealthy and learn how to explore logs in Kubernetes to troubleshoot issues. You will then fix the problem and redeploy the service.

1.  From the Kubernetes dashboard view the logs of the healthy API service and note that it is listening on port 3001.

2.  View the logs for the "web" service and note that there are no indications of the issue.

3.  View the Replica Sets of the unhealthy "web" service and note the Events list indicates warnings of failed health checks. Note that the health check is querying port 80 which is incorrect.

4.  Edit the deployment. Scroll down to view the Port Definitions used when you published the "web" application. Note the container port is incorrectly set to 80, it should be 3000. Therefore, the health check is failing. The public host port is set to correctly to port 80.

5.  Set the containerPort and livenessProbe entries to 3000.

6.  Redeploy the service with these changes by first scaling the "web" service pods to 0, then scaling back to 1.

7.  On the Replication Sets page and you'll see that the "web" pod is replaced. When it is fully initialized, the pod should run normally with no restarts.

### Task 5: Test the application in a browser

In this task, you will verify that you can browse to the "web" service you have deployed, and view the speaker and content information exposed by the API service.

1.  From the Kubernetes management dashboard, navigate to services, find the endpoint for the web service and launch the application in the browser.

2.  You will see the web application in your browser and be able to select the Speakers and Sessions links to view those pages without errors. The lack of errors means that the web service is correctly calling the API service to show the details on each of those pages.

## Exercise 3: Scale the application and test HA

**Duration**: 20 minutes

At this point, you have deployed a single instance of the web and API service containers. In this exercise, you will increase the number of container instances for the API service and scale the front end on the existing cluster.

### Task 1: Increase service instances from the Kubernetes dashboard

In this task, you will increase the number of instances for the API deployment in the Kubernetes management dashboard. While it is deploying, you will observe the changing status.

1.  From the Kubernetes dashboard, scale the number of pods for the API deployment to 2.

2.  Go to the Replica Sets view of the API and note that there is a pending instance deploying.

3.  Go to the Services view and note that the API is pending.

4.  Go to the Workloads view and by this time note that the API and web services are healthy.

5.  Navigate to the web application in the browser to view the content at /stats.html and take note of the following details as you refresh the page:

    -   webTaskId: the task identifier for the web service instance

    -   taskId: the task identifier for the API service instance

    -   hostName: the host name identifier for the API service instance

    -   pid: the process id for the API service instance

    -   mem: some memory indicators returned from the API service instance

    -   counters: counters for the service itself, as returned by the API service instance

    -   uptime: the up time for the API service

6.  The taskId and hostName for the API service will change as you refresh since there are two pods.

### Task 2: Increase service instances beyond available resources

In this task, you will try to increase the number of instances for the API service container beyond available resources in the cluster. You will observe how Kubernetes handles this condition and correct the problem.

1.  Navigate to the API deployment and scale it to four pods.

2.  From the API deployments view note that four pods are running and one pending.

3.  Navigate to the Workloads view and note that there are errors in the API deployment. These errors are due to the fixed port requirement. Only two instances of the API pod can be deployed as there are only two nodes with port 3001 available.

4.  Scale the API deployment back down to 2 pods and note that the errors are almost immediately resolved.

### Task 3: Restart containers and test HA

In this task, you will restart containers and validate that the restart does not impact the running service. First, you will restart the API service, followed by the web service. You will also note that there are issues with HA restarts when there is only a single instance.

1.  Navigate to the Services view and select the external endpoint for the web service. Add /stats.html to the URL to browse to the stats page of the web application.

2.  Navigate to the API deployment and scale it to 4 pods.

3.  Navigate to the API Replica Set view and note that two pods cannot deploy.

4.  Return to the browser tab with the web application stats page loaded. Refresh the page over and over. You will not see any errors, but you will see the API hostName change between the two API pod instances periodically.

5.  Return to the API Replica Set view and note that as you refresh the page, the hostName from the stats page switches between the pod names for the API pods that are running.

6.  Note the remaining pods are still pending, since there are not enough port resources available to launch another instance. Make some room by deleting a running pod instance.

7.  Note that a new pod instance is deployed and, since the desired state is four pods, another pending instance is created.

8.  Scale the API deployment down to 1 pod. As it scales down, return to the web application stats page and refresh, noting that only one API hostName and taskId displays. Kubernetes has already stopped sending requests to the pods that are scaling down.

9.  Navigate to the Workloads view and note that eventually there is only one API pod deployed.

## Exercise 4: Setup load balancing and service discovery

**Duration**: 45 minutes

Until now there have been some restrictions on the scale properties of the service. In this exercise, you will configure a replication controller to create pods that use dynamic port mappings to eliminate the port resource constraint during scale activities.

Kubernetes services can discover the ports assigned to each pod, allowing you to run multiple instances of the pod on the same agent node --- something that is not possible when you configure a specific static port (such as 3001 for the API service).

### Task 1: Create a public load balancer for a service

Any external access to Kubernetes pods requires a load balancer. Because of this, you defined the web service with the type LoadBalancer in the YAML description, and you can access the web application using the public IP. In this task, you will change the API service type to LoadBalancer. When you change the type of the service to LoadBalancer the ACS will create a public-facing load balancer with a public IP address.

1.  Navigate to the API service and edit the service description so that the type setting changes from ClusterIP to LoadBalancer.

2.  Navigate to Services and discovery and under the Services list wait for the API service to show as ready after this deployment is reflected.

3.  The API service will now have an external endpoint. Select the hyperlink and navigate to the /stats path to view the JSON response.

### Task 2: Scale a service without port constraints

In this task, we will reconfigure the deployment's container template so that it will produce pods that choose a dynamic host port.

1.  Edit the API deployment. Set replicas to 4 and remove the hostPort entry.

2.  This time the deployment will be able to run 4 pods since the hostPort restriction has been removed.

3.  Refresh the stats endpoint for the API service deployment in the browser and note that the hostPort entry will start to change as you refresh, to match the names of the 4 pods.

### Task 3: Update a service to support dynamic service discovery without a load balancer

The API service does not need a public endpoint to take advantage of the dynamic port discovery. Earlier in this lab you added a public load balancer with an external endpoint to directly observe dynamic discovery to learn about constraints. Now you will restore the API service to its original configuration in preparation for the following tasks that show off how seamlessly Kubernetes supports scaling services for internal endpoints as well.

1.  Edit the API service deployment. Change the *type* from LoadBalancer to ClusterIP and remove the *nodePort* setting.

2.  When the deployment update is complete and the API service is ready, you will no longer be able to access /stats from the public endpoint. You can browse to the web application path at /stats.html as before and observe hostName and taskId changes to reflect the 4 API pods being called as you refresh.

### Task 4: Update an external service to support dynamic discovery with a load balancer

In this task, you will update the web service so that it supports dynamic discovery through the Azure load balancer.

1.  Edit the web deployment. Remove the *hostPort* setting to allow a dynamic port assignment as pods are created.

2.  Scale the web deployment to 4 pods.

3.  Refresh the web Deployments view and note the error indicating that the deployment cannot scale beyond 2 pod instances. This is due to a CPU constraint since the web service is asking for more CPU than is available. The port constraint has already been resolved by allowing dynamic discovery.

### Task 5: Adjust CPU constraints to improve scale

In this task you will modify the CPU requirements for the web service so that it can scale out to more instances.

1.  Edit the web deployment. Set the *cpu* resource requirement to 125m.

2.  Navigate to the web replica set and note that the four pods will show as deployed and running health.

3.  Return to the browser tab with the web application loaded and refresh the stats page at /stats.html to watch the display update to reflect the different API pod names for the hostName.

### Task 6: Perform a rolling update 

In this task, you will edit the web application source code and update the Docker image used by the deployment. Earlier you scaled down to 0 and then increased scale to update a deployment. This time you will perform a rolling update to demonstrate how a code change is deployed in a production scenario.

1.  Connect to the build agent VM and navigate to the Fabrikam\\content-web directory where the web application source resides.

2.  Using Vim, edit server.js to change process.pid to process.env.HOSTNAME as shown here:

    119. data.webTaskId = process.env.HOSTNAME;

3.  Rebuild the docker image and tag the new image with v2. Push the updated image latest and v2 tags to the ACR repository.

4.  From WSL, request a rolling update using this kubectl command:

    ```
    kubectl set image deployment/web web=\[LOGINSERVER\]/fabmedical/content-web:v2
    ```

5.  Navigate to the web replica set and note the new deployment in progress.

6.  Browse to the web application stats page and note that the webTaskId now updates to show different pod names as you refresh the page.

## After the hands-on lab 

**Duration**: 10 minutes

In this exercise, you will deprovision any Azure resources that were created in support of this lab.

1.  Delete both of the Resource Groups in which you placed all of your Azure resources.

    -   From the Portal, navigate to the blade of your Resource Group and select Delete in the command bar at the top.

    -   Confirm the deletion by re-typing the resource group name and selecting Delete.

2.  Delete the Service Principal created on Task 9: Create a Service Principal before the hands-on lab.

```
az ad sp delete --id "Fabmedical-sp"
```

You should follow all steps provided *after* attending the hands-on lab.

