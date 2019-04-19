2. Use oc new-app to Create Application
In this section, you use the oc new-app command to create an application from container images and source code.

oc new-app is a very powerful command and there are many ways you can use it. As always, use the --help flag in the OpenShift CLI to find out more (oc new-app --help).
Connect to your client VM as described above.

Log in to the OpenShift cluster as described above.

Create a project in which to create applications:

Because project names in OpenShift are unique across the entire cluster, make sure to replace xyz with your initials.

oc new-project xyz-new-apps
Sample Output
Now using project "xyz-new-apps" on server "https://master.na39.openshift.opentlc.com:443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app centos/ruby-22-centos7~https://github.com/openshift/ruby-ex.git

to build a new example application in Ruby.
Create a new application using the latest wildfly image available on Docker Hub:

oc new-app docker.io/jboss/wildfly:latest
Sample Output
--> Found Docker image ec52433 (2 weeks old) from docker.io for "docker.io/jboss/wildfly:latest"

    * An image stream will be created as "wildfly:latest" that will track this image
    * This image will be deployed in deployment config "wildfly"
    * Port 8080/tcp will be load balanced by service "wildfly"
      * Other containers can access this service through the hostname "wildfly"

--> Creating resources ...
    imagestream "wildfly" created
    deploymentconfig "wildfly" created
    service "wildfly" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/wildfly'
    Run 'oc status' to view your app.
Examine the generated OpenShift objects:

oc get all
Sample Output
NAME                   DOCKER REPO                                             TAGS      UPDATED
is/wildfly   docker-registry.default.svc:5000/xyz-new-apps/wildfly   latest    45 seconds ago

NAME                        REVISION   DESIRED   CURRENT   TRIGGERED BY
dc/wildfly   1          1         1         config,image(wildfly:latest)

NAME                 READY     STATUS    RESTARTS   AGE
po/wildfly-1-46hq2   1/1       Running   0          43s

NAME           DESIRED   CURRENT   READY     AGE
rc/wildfly-1   1         1         1         45s

NAME          CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
svc/wildfly   172.30.233.213   <none>        8080/TCP   45s
Review the various objects that the oc new-app command created:

imagestreams/wildfly is the image stream representing the image

deploymentconfigs/wildfly is the deployment configuration representing the OpenShift application

wildfly-1-46hq2 is the pod running the container image

rc/wildfly-1 is the replication controller controlling the pod

svc/wildfly is the service representing the service under which the pods are available

Observe also that the replication controller specifies the following:

Desired is the number of pods that should be running

Current is the number of pods that are currently running

Ready is the number of pods that are ready to receive traffic

Services are used for internal OpenShift communication. This way, another application does not need to know the actual IP address of the pod or how many pods are running for any given application. But since services are not accessible outside the OpenShift cluster, you need to create a route to expose the service to the outside world:

oc expose svc wildfly
Determine what route was created:

oc get route
Sample Output
NAME      HOST/PORT                                           PATH      SERVICES   PORT       TERMINATION   WILDCARD
wildfly   wildfly-xyz-new-apps.apps.na39.openshift.opentlc.com             wildfly    8080-tcp                 None
Use the route to try to access the web server:

curl wildfly-xyz-new-apps.apps.na39.openshift.opentlc.com
Make sure to replace xyz with your actual route.

Sample Output
<!--
  ~ JBoss, Home of Professional Open Source.
  ~ Copyright (c) 2014, Red Hat, Inc., and individual contributors
  ~ as indicated by the @author tags. See the copyright.txt file in the
  ~ distribution for a full listing of individual contributors.
  ~
  ~ This is free software; you can redistribute it and/or modify it
  ~ under the terms of the GNU Lesser General Public License as
  ~ published by the Free Software Foundation; either version 2.1 of
  ~ the License, or (at your option) any later version.
  ~
  ~ This software is distributed in the hope that it will be useful,
  ~ but WITHOUT ANY WARRANTY; without even the implied warranty of
  ~ MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU
  ~ Lesser General Public License for more details.
  ~
  ~ You should have received a copy of the GNU Lesser General Public
  ~ License along with this software; if not, write to the Free
  ~ Software Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA
  ~ 02110-1301 USA, or see the FSF site: http://www.fsf.org.
  -->
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">

<html>
<head>
    <title>Welcome to WildFly 11</title>
    <link rel="shortcut icon" href="favicon.ico" type="image/x-icon">
    <link rel="StyleSheet" href="wildfly.css" type="text/css">
</head>

<body>
<div class="wrapper">
    <div class="content">
        <div class="logo">
                <img src="wildfly_logo.png" alt="WildFly 11... it's here." border="0" />
        </div>
        <h1>Welcome to WildFly 11</h1>

        <h3>Your WildFly 11 is running.</h3>

        <p><a href="documentation.html">Documentation</a> | <a href="http://github.com/wildfly/quickstart">Quickstarts</a> | <a href="/console">Administration
            Console</a> </p>

        <p><a href="http://wildfly.org">WildFly Project</a> |
            <a href="https://community.jboss.org/en/wildfly">User Forum</a> |
            <a href="https://issues.jboss.org/browse/WFLY">Report an issue</a></p>
        <p class="logos"><a href="http://jboss.org"><img src="jbosscommunity_logo_hori_white.png" alt="JBoss and JBoss Community" width=
                "195" height="37" border="0"></a></p>

        <p class="note">To replace this page simply deploy your own war with / as its context path.<br />
            To disable it, remove the "welcome-content" handler for location / in the undertow subsystem.</p>
    </div>
</div>
</body>
</html>
Next, examine the deployment configuration that was created:

oc describe dc wildfly
Sample Output
Name:		wildfly
Namespace:	xyz-new-apps
Created:	9 minutes ago
Labels:		app=wildfly
Annotations:	openshift.io/generated-by=OpenShiftNewApp
Latest Version:	1
Selector:	app=wildfly,deploymentconfig=wildfly
Replicas:	1
Triggers:	Config, Image(wildfly@latest, auto=true)
Strategy:	Rolling
Template:
Pod Template:
  Labels:	app=wildfly
		deploymentconfig=wildfly
  Annotations:	openshift.io/generated-by=OpenShiftNewApp
  Containers:
   wildfly:
    Image:		docker.io/jboss/wildfly@sha256:b519027910032d3951b94412887fc274c1e651b0fe999054c6e528cc9b48a9ea
    Port:		8080/TCP
    Environment:	<none>
    Mounts:		<none>
  Volumes:		<none>

Deployment #1 (latest):
	Name:		wildfly-1
	Created:	9 minutes ago
	Status:		Complete
	Replicas:	1 current / 1 desired
	Selector:	app=wildfly,deployment=wildfly-1,deploymentconfig=wildfly
	Labels:		app=wildfly,openshift.io/deployment-config.name=wildfly
	Pods Status:	1 Running / 0 Waiting / 0 Succeeded / 0 Failed

Events:
  FirstSeen	LastSeen	Count	From				SubObjectPath	Type		Reason			Message
  ---------	--------	-----	----				-------------	--------	------			-------
  9m		9m		1	deploymentconfig-controller			Normal		DeploymentCreated	Created new replication controller "wildfly-1" for version 1
Note the various properties that the oc new-app command set as defaults for this application, including the number of replicas (1) and deployment strategy (Rolling), and at the very bottom of the page, the associated events for this deployment configuration.

Also note that a label (app=wildfly) was created. This label is the same for every object that oc new-app creates during the process.

Examine the pod:

oc describe pod wildfly-1-46hq2
Your pod name will be different. Use oc get pod to determine the actual pod name.

Sample Output
Name:		wildfly-1-46hq2
Namespace:	xyz-new-apps
Node:		node1.na39.internal/192.199.0.50
Start Time:	Wed, 20 Dec 2017 16:28:39 -0500
Labels:		app=wildfly
		deployment=wildfly-1
		deploymentconfig=wildfly
Annotations:	kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicationController","namespace":"xyz-new-apps","name":"wildfly-1","uid":"bd66779a-e5cc-11e7-bb36-0a21d5...
		kubernetes.io/limit-ranger=LimitRanger plugin set: cpu, memory request for container wildfly; cpu, memory limit for container wildfly
		openshift.io/deployment-config.latest-version=1
		openshift.io/deployment-config.name=wildfly
		openshift.io/deployment.name=wildfly-1
		openshift.io/generated-by=OpenShiftNewApp
		openshift.io/scc=restricted
Status:		Running
IP:		10.1.6.195
Created By:	ReplicationController/wildfly-1
Controlled By:	ReplicationController/wildfly-1
Containers:
  wildfly:
    Container ID:	docker://34ffb4f710f8f26b3ca366cf6f1ee14c73d4d992831b5e32c0d9d2ed27665d07
    Image:		docker.io/jboss/wildfly@sha256:b519027910032d3951b94412887fc274c1e651b0fe999054c6e528cc9b48a9ea
    Image ID:		docker-pullable://docker.io/jboss/wildfly@sha256:b519027910032d3951b94412887fc274c1e651b0fe999054c6e528cc9b48a9ea
    Port:		8080/TCP
    State:		Running
      Started:		Wed, 20 Dec 2017 16:28:58 -0500
    Ready:		True
    Restart Count:	0
    Limits:
      cpu:	500m
      memory:	512Mi
    Requests:
      cpu:		50m
      memory:		256Mi
    Environment:	<none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-q5fs6 (ro)
Conditions:
  Type		Status
  Initialized 	True
  Ready 	True
  PodScheduled 	True
Volumes:
  default-token-q5fs6:
    Type:	Secret (a volume populated by a Secret)
    SecretName:	default-token-q5fs6
    Optional:	false
QoS Class:	Burstable
Node-Selectors:	env=users
Tolerations:	<none>
Events:
  FirstSeen	LastSeen	Count	From				SubObjectPath			Type		Reason			Message
  ---------	--------	-----	----				-------------			--------	------			-------
  14m		14m		1	default-scheduler						Normal		Scheduled		Successfully assigned wildfly-1-46hq2 to node1.na39.internal
  14m		14m		1	kubelet, node1.na39.internal					Normal		SuccessfulMountVolume	MountVolume.SetUp succeeded for volume "default-token-q5fs6"
  14m		14m		1	kubelet, node1.na39.internal	spec.containers{wildfly}	Normal		Pulling			pulling image "docker.io/jboss/wildfly@sha256:b519027910032d3951b94412887fc274c1e651b0fe999054c6e528cc9b48a9ea"
  13m		13m		1	kubelet, node1.na39.internal	spec.containers{wildfly}	Normal		Pulled			Successfully pulled image "docker.io/jboss/wildfly@sha256:b519027910032d3951b94412887fc274c1e651b0fe999054c6e528cc9b48a9ea"
  13m		13m		1	kubelet, node1.na39.internal	spec.containers{wildfly}	Normal		Created			Created container
  13m		13m		1	kubelet, node1.na39.internal	spec.containers{wildfly}	Normal		Started			Started container
Examine the output and note again the label (app=wildfly) and especially the events toward the bottom of the output.

These events are crucial in debugging application start problems. For example, if your container image requires running with the root user, OpenShift does not allow the pod to start due to security restrictions. The events log displays any error messages.

Use the label to delete the application:

oc delete all -lapp=wildfly
Sample Output
imagestream "wildfly" deleted
deploymentconfig "wildfly" deleted
route "wildfly" deleted
pod "wildfly-1-46hq2" deleted
service "wildfly" deleted
Note that the route inherited the label from the service and therefore was deleted as well.

3. Use oc new-app to Create Application from Source Code
When you use oc new-app with a source code repository, it tries to examine the source code and determine the correct Source-2-Image (S2I) builder image.

In this section, you use a Node.JS source code repository. Since this repository contains a file called app.js, which is one of the trigger file names, oc new-app automatically selects the Node.JS builder image and creates a build configuration to build the container image from source code.

The Creating New Applications section of the OpenShift Documentation has a helpful table that outlines the trigger filenames for specific builder images.
Create a new application from the source code repository:

oc new-app https://github.com/openshift/nodejs-ex.git
Sample Output
--> Found image 7191d41 (3 weeks old) in image stream "openshift/nodejs" under tag "6" for "nodejs"

    Node.js 6
    ---------
    Node.js 6 available as docker container is a base platform for building and running various Node.js 6 applications and frameworks. Node.js is a platform built on Chrome's JavaScript runtime for easily building fast, scalable network applications. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient, perfect for data-intensive real-time applications that run across distributed devices.

    Tags: builder, nodejs, nodejs6

    * The source repository appears to match: nodejs
    * A source build using source code from https://github.com/openshift/nodejs-ex.git will be created
      * The resulting image will be pushed to image stream "nodejs-ex:latest"
      * Use 'start-build' to trigger a new build
    * This image will be deployed in deployment config "nodejs-ex"
    * Port 8080/tcp will be load balanced by service "nodejs-ex"
      * Other containers can access this service through the hostname "nodejs-ex"

--> Creating resources ...
    imagestream "nodejs-ex" created
    buildconfig "nodejs-ex" created
    deploymentconfig "nodejs-ex" created
    service "nodejs-ex" created
--> Success
    Build scheduled, use 'oc logs -f bc/nodejs-ex' to track its progress.
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/nodejs-ex'
    Run 'oc status' to view your app.
If the source code repository does not have the correct file names, you can tell the oc new-app command which builder image to use by including the builder image name before the repository: oc new-app nodejs~https://github.com/openshift/nodejs-ex.git.
Examine the objects that the oc new-app command created:

oc get all
Sample Output
NAME                     TYPE      FROM      LATEST
bc/nodejs-ex   Source    Git       1

NAME                 TYPE      FROM          STATUS    STARTED          DURATION
builds/nodejs-ex-1   Source    Git@debf979   Running   40 seconds ago

NAME                     DOCKER REPO                                               TAGS      UPDATED
is/nodejs-ex   docker-registry.default.svc:5000/xyz-new-apps/nodejs-ex

NAME                          REVISION   DESIRED   CURRENT   TRIGGERED BY
dc/nodejs-ex   0          1         0         config,image(nodejs-ex:latest)

NAME                   READY     STATUS    RESTARTS   AGE
po/nodejs-ex-1-build   1/1       Running   0          40s

NAME            CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
svc/nodejs-ex   172.30.221.116   <none>        8080/TCP   40s
Expect to see the same objects as before and additionally a build configuration that contains the information OpenShift needs to build this application as well as the actual build that built the container image.

Take a moment to examine the build configuration:

oc describe bc nodejs-ex
Sample Output
Name:		nodejs-ex
Namespace:	xyz-new-apps
Created:	10 minutes ago
Labels:		app=nodejs-ex
Annotations:	openshift.io/generated-by=OpenShiftNewApp
Latest Version:	1

Strategy:	Source
URL:		https://github.com/openshift/nodejs-ex.git
From Image:	ImageStreamTag openshift/nodejs:6
Output to:	ImageStreamTag nodejs-ex:latest

Build Run Policy:	Serial
Triggered by:		Config, ImageChange
Webhook GitHub:
	URL:	https://master.na39.openshift.opentlc.com:443/apis/build.openshift.io/v1/namespaces/xyz-new-apps/buildconfigs/nodejs-ex/webhooks/-0eH6lgGO4YzRoSvGWlY/github
Webhook Generic:
	URL:		https://master.na39.openshift.opentlc.com:443/apis/build.openshift.io/v1/namespaces/xyz-new-apps/buildconfigs/nodejs-ex/webhooks/rCGqfofzj4vNQnGkAGLi/generic
	AllowEnv:	false

Build		Status		Duration	Creation Time
nodejs-ex-1 	complete 	1m0s 		2017-12-20 16:55:41 -0500 EST

Events:
  FirstSeen	LastSeen	Count	From			SubObjectPath	Type		Reason				Message
  ---------	--------	-----	----			-------------	--------	------				-------
  10m		10m		1	buildconfig-controller			Warning		BuildConfigInstantiateFailed	gave up on Build for BuildConfig xyz-new-apps/nodejs-ex (0) due to fatal error: the LastVersion(1) on build config xyz-new-apps/nodejs-ex does not match the build request LastVersion(0)
Note the following:

The app=nodejs-ex label assigned to all created resources

The build strategy: Source

The URL of the repository

The builder image used to build the image: ImageStreamTag openshift/nodejs:6

The target image for the built container image: ImageStreamTag nodejs-ex:latest

The two web hooks, Webhook GitHub and Webhook Generic, that can be used to automatically trigger this build whenever a developer pushes new content into the repository. (Setting this up is beyond the scope of this lab.)

The build events—in this instance you may see a warning that did not affect the build.

3.1. Review Build
Builds are executed in a build pod. You can examine the build logs either by examining the logs of the pod or by asking for the build logs directly.

Find out the name of the latest build:

oc get build
Sample Output
NAME          TYPE      FROM          STATUS     STARTED          DURATION
nodejs-ex-1   Source    Git@debf979   Complete   16 minutes ago   1m0s
Display the build logs for this build:

oc logs build/nodejs-ex-1
Sample Output
Command "build-logs" is deprecated, use "oc logs build/<build-name>" instead.
Cloning "https://github.com/openshift/nodejs-ex.git" ...
	Commit:	debf979e0d6724889c947c13a382785620c18038 (Merge pull request #156 from jim-minter/issue17649)
	Author:	Ben Parees <bparees@users.noreply.github.com>
	Date:	Wed Dec 6 23:18:36 2017 -0500
---> Installing application source ...
---> Building your Node application from source
npm WARN deprecated to-iso-string@0.0.2: to-iso-string has been deprecated, use @segment/to-iso-string instead.
npm WARN deprecated jade@0.26.3: Jade has been renamed to pug, please install the latest version of pug instead of jade
npm WARN deprecated minimatch@0.3.0: Please update to minimatch 3.0.2 or higher to avoid a RegExp DoS issue
nodejs-ex@0.0.1 /opt/app-root/src
+-- chai@3.5.0
| +-- assertion-error@1.0.2
| +-- deep-eql@0.1.3
| | `-- type-detect@0.1.1
| `-- type-detect@1.0.0

[.....]

+-- morgan@1.9.0
| +-- basic-auth@2.0.0
| `-- on-headers@1.0.1
`-- object-assign@4.1.0
Pushing image docker-registry.default.svc:5000/xyz-new-apps/nodejs-ex:latest ...
Pushed 0/6 layers, 2% complete
Pushed 1/6 layers, 28% complete
Pushed 2/6 layers, 45% complete
Pushed 3/6 layers, 60% complete
Pushed 4/6 layers, 77% complete
Pushed 5/6 layers, 100% complete
Pushed 6/6 layers, 100% complete
Push successful
Note the following:

The Node.JS build fetches all dependencies and then pushes the image to docker-registry.default.svc:5000/xyz-new-apps/nodejs-ex:latest.

docker-registry is your integrated OpenShift container registry. The builder is using the internal service name to connect to the registry.

The image is project scoped (xyz-new-apps).

3.2. Review Image Stream
Every image is tracked in an image stream. Remember that the image stream was created by the oc new-app command. But, rather than pointing to an external image on Docker Hub, it is now pointing to your internal container registry where the build pushed the image to. Also remind yourself of the build configuration which had a target image configured.

Examine the image stream:

oc describe is nodejs-ex
Sample Output
Name:			nodejs-ex
Namespace:		xyz-new-apps
Created:		22 minutes ago
Labels:			app=nodejs-ex
Annotations:		openshift.io/generated-by=OpenShiftNewApp
Docker Pull Spec:	docker-registry.default.svc:5000/xyz-new-apps/nodejs-ex
Image Lookup:		local=false
Unique Images:		1
Tags:			1

latest
  no spec tag

  * docker-registry.default.svc:5000/xyz-new-apps/nodejs-ex@sha256:b6f9fd5a255e122cd94eb6c53c91cdd11d88f8b4a9e30647dc625c64984e957c
      21 minutes ago
Note that the image stream has the same label as all of the other objects. Also note that you only have one image tag at the moment, (latest), which points to the unique identifier for the image you just pushed to the registry.

Add a tag to this image so that you can refer to it later even if there are subsequent builds that update the latest tag:

oc tag nodejs-ex:latest nodejs-ex:testing
This command creates the testing tag and sets it to the same image as the latest tag.

Examine the image stream again and note that there are now two tags:

oc describe is nodejs-ex
Sample Output
Name:			nodejs-ex
Namespace:		xyz-new-apps
Created:		26 minutes ago
Labels:			app=nodejs-ex
Annotations:		openshift.io/generated-by=OpenShiftNewApp
Docker Pull Spec:	docker-registry.default.svc:5000/xyz-new-apps/nodejs-ex
Image Lookup:		local=false
Unique Images:		1
Tags:			2

latest
  no spec tag

  * docker-registry.default.svc:5000/xyz-new-apps/nodejs-ex@sha256:b6f9fd5a255e122cd94eb6c53c91cdd11d88f8b4a9e30647dc625c64984e957c
      25 minutes ago

testing
  tagged from nodejs-ex@sha256:b6f9fd5a255e122cd94eb6c53c91cdd11d88f8b4a9e30647dc625c64984e957c

  * docker-registry.default.svc:5000/xyz-new-apps/nodejs-ex@sha256:b6f9fd5a255e122cd94eb6c53c91cdd11d88f8b4a9e30647dc625c64984e957c
      About a minute ago
Expect to see two tags: latest and testing, but still only one unique image (you can verify by comparing the sha256 hash codes for both tags).

Next, start a new build to push a new version of the image into the registry:

oc start-build nodejs-ex
Tail the logs of the build once it is running (verify with oc get pod until you see a build pod running):

oc logs -f nodejs-ex-2-build
Once the image has been pushed successfully (expect to see that only layer 5 and 6 were actually updated), re-examine your image stream:

oc describe is nodejs-ex
Sample Output
Name:			nodejs-ex
Namespace:		xyz-new-apps
Created:		30 minutes ago
Labels:			app=nodejs-ex
Annotations:		openshift.io/generated-by=OpenShiftNewApp
Docker Pull Spec:	docker-registry.default.svc:5000/xyz-new-apps/nodejs-ex
Image Lookup:		local=false
Unique Images:		2
Tags:			2

latest
  no spec tag

  * docker-registry.default.svc:5000/xyz-new-apps/nodejs-ex@sha256:220ea932d3d32f31a5fb533d9df3e54d7c38b0555b1e80beeab0810a511d25c9
      About a minute ago
    docker-registry.default.svc:5000/xyz-new-apps/nodejs-ex@sha256:b6f9fd5a255e122cd94eb6c53c91cdd11d88f8b4a9e30647dc625c64984e957c
      29 minutes ago

testing
  tagged from nodejs-ex@sha256:b6f9fd5a255e122cd94eb6c53c91cdd11d88f8b4a9e30647dc625c64984e957c

  * docker-registry.default.svc:5000/xyz-new-apps/nodejs-ex@sha256:b6f9fd5a255e122cd94eb6c53c91cdd11d88f8b4a9e30647dc625c64984e957c
      5 minutes ago
Expect to see two unique images, and that the latest tag points to a different image than the testing tag (the sha256 hash codes are different).

4. Clean Up Environment
To remove your work and save resources for future labs, delete the project:

oc delete project xyz-new-apps
