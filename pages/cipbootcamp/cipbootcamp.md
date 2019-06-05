---
title: CIP Bootcamp
toc: false
sidebar: labs_sidebar
folder: pots/cip-bootcamp
permalink: /cip-bootcamp-lab.html
summary: Introduction to the Cloud Integration Platform
applies_to: [developer,administrator,consumer]
---

# Getting Started
===============

## Familiarization with the Cloud Integration Platform

The Cloud Integration Platform (CIP) is built on top of IBM Cloud Private.  This lab guide is meant to give you an overview of the capabilities of the platform, and get some hands on time with setting up your own environment for the purpose of creating your own demo environment.

This lab guide assumes some familiarity with IBM Cloud Private, Docker & Kubernetes, as well as Helm.  

There are two primary components of CIP:

**IBM Cloud Private (ICP)** -  An application platform for developing and managing on-premises, containerized applications. It is an integrated environment for managing containers that includes the container orchestrator Kubernetes, a private image repository, a management console, and monitoring frameworks. In this particular boot camp, ICP is deployed on Softlayer, using Skytap as the primary operational interface.

**Platform Navigator** --  an application that is built to run on top of IBM Cloud Private that provides a primary user interface where instances of App Connect Enterprise (ACE), API Connect (APIC), Event Streams and MQ Advanced can be managed from a single place.  It is automatically deployed with the CIP Bundle.

As a convention for these labs, a <span style="color:red"> **red box** </span> will be used to identify a particular area, and when information is to be entered and/or an action is to be taken, it will be in **bold** characters. <span style="color:red"> **Red arrows or lines** </span> may be used to indicate specific items that need to be noted as part of the lab

A number of sections will be labeled “Key Idea” or “Key Concept”. Be sure to review these sections when you come across them, as they will  explain important items as it relates to the lab content.

Lab Overview
-------------------------------------------

This lab is broken up into two key parts. 

**Part One**:  Create and Deploy your CIP Applications

**Part Two**:  Deploy Integration Assets


Part one contains the hands on section of the lab where you will get acquainted with the Platform Navigator and be setting up the individual components of the platform.

Part two you will be deploying some basic assets to your newly created environment


Part One: Create and Deploy your CIP Applications
-------------------------------------------

1.  You have been provided a pre-installed environment of the Cloud Integration Platform.  It includes a vanilla ICP 3.1.1 install with the CIP Platform Navigator installed and all of the base container images that make up CIP.
2. The environment you are using consists of 9 different nodes.  8 of which are ICP Nodes and one is a developer image that you will be using to access the ICP User Interfaces.  You can access this VM directly using the Skytap interface to use the X-Windows based components.  
3. **VERY IMPORTANT** It is very important you do not suspend your lab environment.  We have seen cases when the environment goes into suspend mode, the Rook-Ceph shared storage gets corrupted.  Also it is a good practice that you shut down ICP before powering down your enviroment.  A script has been included to handle that for you that will be explained in the coming sections.  When you power down your environment, you can safely use the `power off` option as the shared storage can interfere with the normal graceful shutdown method.  So far using power off hasn't caused any problems that we are aware of.
3. You are also able to SSH into your environment.  You can find out the exact address to SSH to in your Skytap Environment window under `networking` and `published services`.  The Published Services set up for you will be for the SSH Port (22) under the `CIP Master` node.  The published service will look something like `services-uscentral.skytap.com:10000`.  Your environment will have a different port on it.  You can then SSH to to the machine from your local machine.  
4. Additionally, you can access the machine direct via the Skytap UI.  This is functional, but can be cumbersome to work with as its not easy to copy and paste into and out of Skytap, especially for the non X-Windows based environments.  
5. Password-less SSH has also been enabled between the Master node and the other nodes in the environment.  **note** a table with the environment configuration is provided below.  Credentials for each machine are `root`/`Passw0rd!`.   

| VM        | Hostname  | IP Address | # of CPU | RAM    | Disk Space (LOCAL) | Shared Storage | Additional Notes |
|-----------|-----------|------------|----------|--------|--------------------|----------------|------------------|
| Master    | master    | 10.0.0.1   | 12       | 32 GB  | 400 GB             | N/A            |                  |
| Worker 1  | worker-1  | 10.0.0.2   | 8        | 16 GB  | 400 GB             | Monitor  | Ceph Master      |
| Worker 2  | worker-2  | 10.0.0.3   | 8        | 16 GB  | 400 GB             | 			  | 		           |
| Worker 3  | worker-3  | 10.0.0.4   | 8        | 16 GB  | 400 GB             | 			  | 		            |
| Worker 4  | worker-4  | 10.0.0.7   | 8        | 16 GB  | 400 GB             |                |                  |
| Worker 5  | worker-5  | 10.0.0.8   | 8        | 16 GB  | 400 GB             | Ceph OSD1               |                  |
| Worker 6  | worker-6  | 10.0.0.9   | 8        | 16 GB  | 400 GB             | Ceph OSD2               |                  |
| Developer | developer | 10.0.0.6   |          |        |                    |                |                  |


6. If it is not done already, power up your Environment.  It could take about 5 minutes for all nodes to start up.  The master node takes the longest to come up, so if you can see the login prompt from the Skytap UI on the master node, then you are good to go. 
7. SSH into the Master Node or use the Skytap UI.  In the home directory of root (`/root`) there is a script called `icpStopStart.sh`.  Run this script by typing in `icpStopStart.sh start` Note that it takes around 30 minutes for the ICP Services to come up completely.
8. You can tell if the start of ICP is complete by checking using one of these two methods:
	- Click on the Developer Machine, and it will take you directly to the Developer Machine running X-Windows.  Should you need to Authenticate, you can use the credentials of `student`/`Passw0rd!`. Bring up the Firefox browser.  Navigate to the main ICP UI by going to `https://10.0.0.1:8443`.  The credentials again are `admin/admin`.  If you can log in and see the main ICP Dashboard, you are good to go.  
	- Alternatively, you can SSH to the Master node.  Execute a `cloudctl login` from the command line.  If all there services are up, it will prompt you for credentials (use `admin`/`admin`) and setup your kubernetes environment.
8. The best place to do your Kubernetes CLI work is from the Master node.  Again, Before you can execute any of the `kubectl` commands you will need to execute a `cloudctl login`. 
9. Next step is to find The Platform Navigator UI can be use to create and manage instances of all of the components that make up the Cloud Integration Platform. **note** at the time of this lab, the ability to create and manage Aspera instances has not yet be added to the Platform Navigator, and will be added at a later date.
10. You can access the Platform Navigator using the browser on the developer machine. The Platform Navigator is bookmarked on the browser.  You might be asked to authenticate into ICP again, but once you do that you should now see the Platform Navigator page.
11. The Platform Navigator is designed for you to easily keep track of your integration toolset.  Here you can see all of the various APIC, Event Streams, MQ and ACE instances you have running.  You can also add new instances using the Platform Navigator.
11. Each application instance requires its own namespace and pull secret for install.  To support this lab, namespaces and pull secrets have been provided for you already, with the exception of Event Streams (ES).  We'll talk more about ES later in the lab.

### Key Concepts - Troubleshooting / Recovery

There are times where things may not be going right, so your best bet is to use `kubectl` commands to uncover more information about what is going on.  If you post queries in Slack about trouble with the lab, we will be asking you to execute a series of these, depending upon what you were doing, and what error occured.  A list of useful commands are provided below:

 - `kubectl get pods -n <some-namespace>` This command shows all of the pods in a given name space.  Here you will see if any pods are up, down, errored or otherwise in transition.
 - `kubectl describe pods <some-pod> -n <some-namespace>` This will provide verbose information about a given pod.  You can use the `describe` command for other objects.  Describe also works with other kubernetes objects like node, pvc, svc etc.
 - `kubectl logs <some-object> -n <some-namespace>` This command will work with other objects as well.  You can also tail the logs by using the `-f` switch at the end.

 Further reference on kubectl commands can be found [here](https://kubernetes.io/docs/reference/kubectl/kubectl-cmds/)

 If you happen to mess up an install of the components, its not difficult to recover.  You can use the ICP UI to remove the release or use the CLI by issuing `helm delete --purge <helm-release-name> --tls`.  Be careful using this though, as this process is not reversable.  Also note that removing API Connect requires additional steps.  These are documented below under the API Connect heading.
 
Logging is also done inside of ICP using the ELK stack.  You can access the logging inside of ICP and use Elastic Search commands to drill into things.  For example, if you were to bring up Kibana and enter in a search like `kubernetes.namespace:<your namespace>`.  Using this along with the `kubectl` commands gives you a lot of power to dig into root cause.
 
 There are plenty other commands to use, but these are by far the most common the author of these labs has used while setting these up :)


## App Connect Enterprise

10. Let's start by creating an App Connect Enterprise Integration Server.  From the Platform Navigator, locate the middle panel for `Application Integration` and click on the `add new instance` link.
11. A pop up window will give you some important information about pre-requisites for creating the instance.  Click `continue`.
12. This will take you to the chart configuration.  If you scroll to the bottom, you can peruse the information about the chart you are about to use to create your instance of App Connect Enteprise.  Click the `Configure` button to configure your chart.  Other than the items listed below, we will be accepting the default values in the chart.
13. **Note** if you get a HTTP 404 when trying to locate your chart.  Try to re-sync your helm repository in ICP by going to the main ICP UI (10.0.0.1:8443) then from the top-left hamburger menu `manage` -> `Helm Repositories`.  Then Click on the `Sync Repositories`.  Try to install the instance again.
13. Here you can fil in the information for install of your chart.  Give your helm release a `name`.  Call the release `ace-icip`.
14. To the right of the `name` field, Select the `ace` namespace from the `Target Namespace` dropdown.
15. Tick checkbox under `license`
16. Expand the `All Parameters` twisty
17. Find the `Image Pull Secret` line.  Set that to `sa-ace` (This is pre-defined for you, normally, you would need to create this).
18. For the `Hostname of the ingress proxy to be configured` section. Leave this as `mycluster.icp`  
18. **IMPORTANT** - Find the `Enable Persistant Storage` Section.  This box must be unticked (disabled).  In order to support persistant storage, Gluster storage must be used (Ceph is not supported with ACE).  Implementing/Using Gluster Storage with CIP beyond the scope of this lab.  [here](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.2/manage_cluster/configure_glusterfs_after.html)
19. Leave the other default values in the chart.  Scroll down and Click `Install` button on lower right portion of the screen.
20. Once you see the `Installation Started` pop up. You can validate the progress via the `Helm Releases` in the ICP Portal or via the command line.
21. Return to your command line and  open up a SSH session to your master node on your laptop or desktop.  **Note** Each environment has a Skytap Published Service set up for every node on port 22 so you can SSH to the nodes from your machine.  If you do a `kubectl get pods -n ace` you should see your pod in a running state.
22. Returning to the Platform Navigator, you should see your new ACE instance there.  Click on it to bring up the ACE Enterprise UI.  **Note**  if you encounter issues with the self signed certs on the browser.  Click on the link where it says `open ace`.  This will open a new browser window and should allow you to access the UI properly.
22. The ACE Dashboard can alternatively be accessed via a web browser and entering the URL for the ACE Dashboard Portal manually. Run the following commands to get the dashboard URL via SSH:

`export ACE_DASHBOARD_URL=$(kubectl get ingress ace-icip-ibm-ace-dashboard-icip-prod-hostname -o jsonpath="{.spec.rules[0].host}{.spec.rules[0].http.paths[0].path}")`

then `echo "Open your web browser to https://${ACE_DASHBOARD_URL}"``

*Hint* If the first command fails, you might not be in the `ace` namespace.  Quickest way to set your context is just to relogin using `cloudctl login` and then set your namespace to `ace`.  

If you can see the App Connect Enterprise portal, then you are are done with this step.  We'll load in a .bar file later.

## MQ

10. Under normal circumstances, you would simply install MQ via the Platform Navigator.  This would setup a plain MQ install with no MQ objects configured.  If you know MQ, you can install via this method, but MQ configuration is beyo
10. For MQ we will not be using the default chart that is linked to the Platform Navigator.  What comes with the CIP is the production version of the chart that requires the installer to configure everything, which is cumbersome if you are just looking to spin up an environment quickly.  Fortunately, ICP comes with a chart that is designed specifically for developer types that comes with a pre-configured queue manager, queues, server channels and the like.
11. To access this chart, make sure you are logged into the ICP UI via the Developer Machine (`https:10.0.0.1:8443`). Using the top menu, right hand side, click on `Catalog`
12. usign the context based search filter - type in `mq`into the filter bar.  Select the `ibm-mqadvanced-server-dev` chart.  You can scroll down a bit to peruse the pre-requisites.  
12. This will take you to the chart configuration.  If you scroll to the bottom, you can peruse the information about the chart you are about to use to create your instance of MQ.  Click the `Configure` button to configure your chart.  Other than the items listed below, we will be accepting the default values in the chart.
13. Here you can fil in the information for install of your chart.  Give your helm release a `name`.  Call the release `mq`.
14. To the right of the `name` field, Select the `acemq` namespace from the `Target Namespace` dropdown.
15. Tick checkbox under `license`
16. Expand the `All Parameters` twisty
18. Under the `Image Pull Secret` section enter in `sa-mq`.
18. Under `Persistence` untick both `Enable Persistence` and `Use Dynamic Provisioning`.  *Note* failure to untick this will cause your install to fail.  Like ACE, MQ requires persistent storage using Gluster if you want to use this feature. As indicated previously, installing Gluster is beyond the scope of this lab.  If you really want to know more about this, it is documented [here](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.2/manage_cluster/configure_glusterfs_after.html)
19. In the `Service Type` section, select `Node Port`.
19. You can leave the other settings as defaults.
20. Click `Install` to start the process.
21. You can monitor the installation from Helm Releases in the ICP UI or via the command line using `kubectl get pods`.  Once you see your pod up and running, you can navigate to the UI via `Workload` ->`Helm releases`.  Find your MQ instance and click on it.  
22. Scroll down a bit and Locate the `Service` section.  Click on the  URL that is called `mq-dev-ibm-mq`
23. Under `Service Details` locate the URL called `console-HTTPs`.  This will start up the MQ Console.  Login with the credentials of `admin/admin`
23. You can now review your pre-configured MQ instance.


## Event Streams

20. Before we create a new Event Streams instance, you will need to delete the existing Event Streams instance and namespace on the environment.
21. There a couple of ways to remove an installation.  We'll do this using in two parts, one by UI, one by CLI. Using the ICP UI (`https://10.0.0.1:8443`) go to the top left hamburger config menu select `Workloads` -> `Helm Releases`.  Locate the `eventstreams` install. *Hint* you can filter releases using the search bar at the top.    Find the `action` icon on the far right (looks like 3 dots).  Click that and then select `Delete`.  ICP will remove all of the Pods that are part of this install.
22. Next we will remove the old namespace. Lets do that via the command line.  Connect to your master node via SSH.  Login via `cloudctl login` if you have to.  Select default namespace.
23. Delete the namespace as follows  `kubectl delete namespace eventstreams`.  Deleting the namespace should be fairly quick.
24. Now lets recreate the namespace.  Do this via `kubectl create namespace eventstreams` 
22. Change context to this new namespace by issuing `kubectl config set-context mycluster-context --user=admin --namespace=eventstreams`
23. re-create your pullsecret then by issuing: `kubectl create secret docker-registry sa-eventstreams --docker-server=mycluster.icp:8500 --docker-username=admin --docker-password=admin --docker-email=admin`
24. Return back to the Developer Machine via Skytap.  Bring up the Platform Navigator again.  Locate the panel on the far right for `Messaging` and click on the `add new instance` link and then select `New Event Streams` instance.
25. Similar to the other instances you have setup, you will have a pop up that gives you some advice on requirements for install.  Click `continue`
26. Like before Click `Continue` in the lower right hand corner. Here we will configure the chart for install.  Other than the items listed below, we will be accepting the default values in the chart.
27. Under `Helm Releases` call it `eventstreams-cip`
28. Set the `Target Namespace` to `eventstreams`
29. Tick the checkbox under `License`
30. Moving down, set the `Image Pull Secret` to `sa-eventstreams`
31. Click the `All Parameters` Twisty
32. Under the Global Install Parameters heading, untick the `Used in Production` checkbox.
33. Modify the `Docker Image Registry` entry by removing the last trailing '/' character.  e.g. `mycluster.icp:8500/icip`.  This is a design feature of this initial release, and will be fixed later.
34. Moving down, make sure all of the persistant storage checkboxes are unticked.
35. That is the last change.  Click the `Install` button on the lower right.
36. Check the progress of the install using the command line (via SSH) to the master node.  `kubectl get pods -n eventstreams`.  Use the `watch` command to monitor the pods so you don't have to keep repeating the same command.
37. If you see all the pods up properly then you are good to go. It should take only a few minutes to spin up all of the required pods.
38. You can access the management interface by returning back to the ICP UI (https://10.0.0.1:8443) on the Developer Machine.  From the main ICP UI, upper left hand corner, click on the hamburger icon and go to `Workloads` -> `Helm Releases`.  Locate your Eventstreams release and then click the `launch` button on the far right. From the drop down list select `admin-ui-https`.  If you see the event streams UI come up, you are then ready to configure event streams

## API Connect Install

20. To install API Connect, there are a few items that are needed in order to properly configure your ICP environment for install.  These are listed below
	- Persistant Storage (Ceph is required, and has been set up for you already)
	- 7 FQDN's for the various UIs/Portal of API Connect
21. Below is a list of the FQDN's you will be using for this configuration.

### API Connect Management ###
**Platform API Endpoint:**   `mgmt.10.0.0.1.nip.io`

**Consumer API Endpoint:**   `mgmt.10.0.0.1.nip.io`

**Cloud Admin UI Endpoint:** `mgmt.10.0.0.1.nip.io`

**API Manager UI Endpoint:** `mgmt.10.0.0.1.nip.io`

### API Connect Portal ###
**Portal Director API Endpoint:** `pd.10.0.0.1.nip.io`

**Portal Web UI Endpoint:**       `pw.10.0.0.1.nip.io`

### API Connect Analytics ###
**Analytics Ingestion API Endpoint:** `ai.10.0.0.1.nip.io`

**Analytics Client API Endpoint:**    `ac.10.0.0.1.nip.io`

### API Connect Gateway ###
**Gateway Service API Endpoint:** aka Management Endpoint `gws.10.0.0.1.nip.io`

**API Gateway Endpoint:** aka API Invocation Endpoint       `apigw.10.0.0.1.nip.io`

21. Return to the Developer Machine and bring up the Platform Navigator.  On the left hand side of the screen locate the `API Lifecycle and Secure Access` section.  At the bottom of this section, locate and click on the `add new instance` link.
21. A pop up will come up, giving you some information about setting up the instance.  Click `Continue`.
22. Click on the `Configuration` heading.  Other than the items listed below, we will be accepting the default values in the chart.
23. Set the `Helm Release` to `apic-cip`.  
24. Moving to the right, set the `Target Namespace` from the dropdown to `apic`
25. Tick the checkbox under `License`
26. Make sure the `Production` tick box is not checked
26. Under the `Parameters` heading.  Set the `Registry Secret` to `sa-apic`.
27. Set `Storage Class` to `rbd-storage-class` This is your Ceph Storage class name. **NOTE** it is very important that this setting is correct, otherwise the setup of the shared storage volumes will fail.  **Note** you can issue a `kubectl get pvc -n apic` once you kick off the install of your chart to see how the shared space is being used by APIC and if there are any problems e.g. if you see an entry in a `pending` state for an extended period of time.
28. Set the `Helm TLS Secret` to `helm-tls-secret`
29. Contuining on, under the `Management` heading.  Here you need to fill in the four different management headings as per the chart in step 2 above.
30. Continue with the same process for the `Portal`, `Analytics` and `Gateway` sections.  Follow the chart above and enter in the values for each of the above values in the FQDN list in Step 2.
31. When done, click the `All Parameters` twisty.
32. Scroll up a bit and start with the values under the `Global` heading.  Select the `Mode` drop down, and set the value to `dev`.
33. Scroll down a bit and located the `Cassandra` heading.  Set that value of `Cluster Size` to the value of `1`.
34. Scroll down some more to the `Gateway` section.  Set the `Replica Count` to the value of `1`.
35. That completes the chart configuration.  Click the `Install` button to start the process.  The Entire Process to install API Connect takes about 15 minutes to spin up all of the pods.  You can monitor things using the watch command e.g. `watch kubectl get pods -n apic`.

### Certificates

In order to access the API Connect environment, you will need to extract the certificates from the created installation and load the main cluster ca-cert into your browser (make sure you are using chrome).  Do this follow these instructions:

Get the root certificate of an APIC instance in namespace my-apic-namespace:

* `kubectl get secrets -n my-apic-namespace | grep apim-velox-certs`
* Copy the secret name, e.g. `apim-velox-certs-4934a84f6b17cf16bfb5b702f2e51d13`
* Then execute `kubectl get secrets apim-velox-certs-4934a84f6b17cf16bfb5b702f2e51d13 -n my-apic-namespace -o json > ~/Desktop/apic-ca-cert.json`.  Note the podname will be different for you, but is included above as a sample.
* Then you'll need to extract the cluster cert located in the json file and base64 decode it.
* copy the cluster ca entry into a new file (cluster.txt) and then `base64 -d cluster.txt > cluster-ca.cert`
* Load the ca cert into chrome via the cert import process.  

Once this is done, you can proceed with the APIC Configuration steps below.

### Troubleshooting

There are a number of ways things might go wrong here.  The good news here is that you can rely on your `kubectl` commands to drill into things.  If, for whatever reason you need to back out an install of API Connect on the platform, these steps *must* be followed:

* `helm delete —purge <yourreleasename> —tls —no-hooks`
* `helm ls --all --namespace apic --tls <install-namespace>` then `helm delete --purge <release> --tls` 
* `kubectl delete deployments,pod,sts,ds,sts,job,pvc -n apic --all`
* `kubectl delete apiconnectclusters --all`
* `kubectl delete secret -l heritage=apicup`

Once these steps are complete, you can proceed with a reinstallation

## API Connect Configuration

20. These next steps are the standard setup steps for configuring a provider organization inside of APIC.  
21. On the Developer Machine, open up the platform navigator and you can access the Cloud Manager by clicking the 'cog' icon to the right of your APIC instance name.
22. Alternatively, you open a new browser tab to `https://mgmt.10.0.0.1.nip.io/admin`.  Once the Admin Portal comes up, you can log in with `admin/8iron-hide`. 
22. You will now be placed into the Cloud Manager Portal.  Click the link to `Configure Topology`
23. Click on `Register Service`
24. Select `Datapower Gateway v5 Compatible`
25. Under Gateway details - give the gateway a name and then under `Management Endpoint` set that value to `https://gws.10.0.0.1.nip.io`
26. Set the `API Invocation endpoint` to ` https://apigw.10.0.0.1.nip.io`.  Click `Save`
28. Once returned back to the Topology screen, click `Register Service` and then select `Analytics`.
29. Set the Title to whatever you like and then change the `management endpoint` to `https://ac.10.0.0.1.nip.io`.  Click `Save`.
30. Once returned back to Topology screen, click `Register Service` then select `Portal`.  *Note* this doesn't create a portal yet - it just registers the service.  We'll add the portal after we create the provider org.
31. Set the Title to whatever you like then change the `Management Endpoint` to `https://pd.10.0.0.1.nip.io`.  
32. Change the `Portal Website URL` `https://pw.10.0.0.1.nip.io`.  Click Save.
33. You can then associate the analytics service to the gateway service by clicking on the `Associate Analytics` link.
34. Setup notifications for your APIC Instance.  Go to `Settings` -> `Notifications`
35. Under Sender & Email Server click `Edit`.
36. Enter in a `Name` and `Email` for your Admin user, this can be anything.
37. Down Below, click on the link that says `Configure Email Server`
38. Set up an email server of your choice.  Suggest using something like `mailtrap.io`, but gmail works also but it might take some configuration as google security has been tightened up on using SMTP on Gmail.
39. From the top left menu - go to `Provider Organizations`.
40. Click `Add` -> `Invite Organization Owner`.  Enter in a email address, use something different here.
41. Once you see the activation link, click on it.  Copy the link and paste it to a new browser tab window on the developer machine.
42. No need to fill out any of the information that pops up, API Connect will grab your credentials from ICP, so going forward the API Manager UI will use SSO facilitated via ICP.
43. You will automatically be logged into the API Manager.  From here you can continue with the next section.



Part Two: Deploy some Integration Assets
-----------------------------

1.  One your developer machine.  Go to a command prompt and execute a `git clone https://github.com/ibm-cloudintegration/techguides`.  
2.  Load up the management page for your ACE install.  You can access it from the Platform Navigator by clicking on your `ace` install or just direct your browser to `https://ace.10.0.0.5.nip.io/ace-ace-cip/`
3. Click on the `+Server` button.  On the local file system, navigate to the `/home/student/techguides` directory and find the lone bar file which should be `RESTRequest_Everything.bar`.  A pop up with a content URL will come up.  Copy this to your clipboard.  You will need it for the next step.
4. You will now deploy your bar file using the ACE chart, so you will be taken to the chart configuration screen in ICP.  Click the `Configure` button or the `Configuration` tab at the top to continue
5. Set the `Helm Release` to `lab`
6. Change the `Target Namespace` to `ace`
7. Tick the `License` checkbox
8. Click on the `All Parameters` twisty to open it up
7. Paste in the URL given to you above `Content Server URL`
8. Untick the `Production Usage` checkbox
9. Set the `Image Pull Secret` to `sa-ace`
10. Untick the `Enable Persistence` and `Use Dynamic Provisioning` checkboxes.
11. Under the `Service` settings, find the `NodePort IP` change this to `lab.10.0.0.5.nip.io`.  This will be the endpoint for your service running on ACE.
12. Leave the remaining settings as defaults and then click `Install` at the bottom.  Your chart will now install.  You can view the progress of the install via Helm Releases as prompted or use `kubectl`.
13. Return back to your ACE Management UI. (Click the `done` on the window from the previous step).  You should now see one service running on your integration server.  Click on the 3 dot hamburger button on the `lab` icon.  Click Open.
14. You will now see 3 icons, one for the REST Request API, Application and Shared Library.  Click on the hamburger icon for REST Request_API and then click open
15. You will then see two URL's that look something (but not exactly) like this:  

	- REST API Base URL `http://lab.10.0.0.1.nip.io:30098/restrequest_api/v1`
	- OpenAPI document `http://lab.10.0.0.1.nip.io:30098/restrequest_api/v1/Tutorial_swagger.json`

Copy and paste the link for the OpenAPI document into a browser window and then save the json file on the local file system.  We will use this inside of API Connect

16. Open up your API Manager Window inside the developer machine via the Platform Navigator.  The login should happen automatically as it will use your ICP credentials for login.
2. Click on the `Manage Catalogs`
3. Click on `Sandbox`
4. Click on `Settings` from left menu
5. Click on `Gateway Services`
6. Click `edit` and choose `Gateway`.
17. From the API Manager homescreen, use the menu on the left to navigate to `Develop`.
18. On the APIs and Products screen click `add` -> `Api`
19. Select the `From Existing API Service` radio button option.  Click `Next`.
20. Click the `browse` button to navigate to the `Tutorial_swagger.json` file on your file system you had saved previously.
21. Click `Next`
22. Review the settings then click `Next` twice (do not activate the API yet).
23. Review the summary to ensure the API was created properly, and then click the `Edit API` button.
24. Click the `Assemble` tab to bring up the Assembly editor.
25. Click the Play button to start the Test tool.
25. Click the blue `Activate API` button to deploy.
26. Select the `get /UserNumber` operation.
27. Scroll down to the `Parameters` section and find the `UserNumber` - replace what is there with the number `1`.
26. Scroll down and click the `Invoke` button.  The first time you invoke this, it will bring up a CORS warning.  Copy and paste the link for the API into a new browser tab.  It will return a 401 error for authorization, but this will load up the cert.
27. Run the invoke again, and you should see the output properly.

### Conclusion
You have completed the initial CIP Labs by taking a base ICP install and then imported some basic integration assets and exposed that as an API running on the configuration.  The key value is having the single platform that runs all of the capability that can be managed easily, user the power of Kubernetes.  

 
**End of Lab**
