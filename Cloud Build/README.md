## Building Containers with DockerFile and Cloud Build

Enable APIs and Services

• Cloud Build
• Container Registry


In Cloud Shell 

Create an empty quickstart.sh file using the nano text editor.

` nano quickstart.sh `

Add the following lines in to the quickstart.sh file:

`#!/bin/sh `

` echo "Hello, world! The time is $(date)." `

Save the file and close nano by pressing the CTRL+X key, then press Y and Enter.

Create an empty Dockerfile file using the nano text editor.

` nano Dockerfile `

Add the following Dockerfile command:

` FROM alpine `

This instructs the build to use the Alpine Linux base image.

Add the following Dockerfile command to the end of the Dockerfile:

` COPY quickstart.sh / `

This adds the quickstart.sh script to the / directory in the image.

Add the following Dockerfile command to the end of the Dockerfile:

` CMD ["/quickstart.sh"] `

This configures the image to execute the /quickstart.sh script when the associated container is created and run.

The Dockerfile should now look like:

` FROM alpine `

` COPY quickstart.sh / `

` CMD ["/quickstart.sh"] `

Save the file and close nano by pressing the CTRL+X key, then press Y and Enter.

In Cloud Shell, run the following command to make the quickstart.sh script executable.

` chmod +x quickstart.sh `

In Cloud Shell, run the following command to build the Docker container image in Cloud Build.

` gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/quickstart-image . `


## Building Containers with a build configuration file and Cloud Build

Cloud Build also supports custom build configuration files. We will incorporate an existing Docker container using a custom YAML-formatted build file with Cloud Build.

In Cloud Shell enter the following command to clone the repository to the lab Cloud Shell.

` git clone https://github.com/GoogleCloudPlatform/training-data-analyst `

Create a soft link as a shortcut to the working directory.

` ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s `

Change to the directory that contains the sample files for this lab.

` cd ~/ak8s/Cloud_Build/a `

A sample custom cloud build configuration file called cloudbuild.yaml has been provided for you in this directory as well as copies of the Dockerfile and the quickstart.sh script you created in the first task.

In Cloud Shell, execute the following command to view the contents of cloudbuild.yaml.

` cat cloudbuild.yaml `

You will see the following:

steps:
- name: 'gcr.io/cloud-builders/docker'
  args: [ 'build', '-t', 'gcr.io/$PROJECT_ID/quickstart-image', '.' ]
images:
- 'gcr.io/$PROJECT_ID/quickstart-image'

This file instructs Cloud Build to use Docker to build an image using the Dockerfile specification in the current local directory, tag it with gcr.io/$PROJECT_ID/quickstart-image ($PROJECT_ID is a substitution variable automatically populated by Cloud Build with the project ID of the associated project) and then push that image to Container Registry.

In Cloud Shell, execute the following command to start a Cloud Build using cloudbuild.yaml as the build configuration file:

` gcloud builds submit --config cloudbuild.yaml . `

The build output to Cloud Shell should be the same as before. When the build completes, a new version of the same image is pushed to Container Registry.

## Building and Testing Containers with a build configuration file and Cloud Build

The true power of custom build configuration files is their ability to perform other actions, in parallel or in sequence, in addition to simply building containers: running tests on your newly built containers, pushing them to various destinations, and even deploying them to Kubernetes Engine. We will see a simple example: a build configuration file that tests the container it built and reports the result to its calling environment.

In Cloud Shell, change to the directory that contains the sample files for this lab.

` cd ~/ak8s/Cloud_Build/b `

As before, the quickstart.sh script and the a sample custom cloud build configuration file called cloudbuild.yaml has been provided for you in this directory. These have been slightly modified to demonstrate Cloud Build's ability to test the containers it has build. 

In Cloud Shell, execute the following command to view the contents of cloudbuild.yaml.

` cat cloudbuild.yaml `

You will see the following:

steps:
- name: 'gcr.io/cloud-builders/docker'
  args: [ 'build', '-t', 'gcr.io/$PROJECT_ID/quickstart-image', '.' ]
- name: 'gcr.io/$PROJECT_ID/quickstart-image'
  args: ['fail']
images:
- 'gcr.io/$PROJECT_ID/quickstart-image

In addition to its previous actions, this build configuration file runs the quickstart-image it has created. In this task, the quickstart.sh script has been modified so that it simulates a test failure when an argument ['fail'] is passed to it.

In Cloud Shell, execute the following command to start a Cloud Build using cloudbuild.yaml as the build configuration file:

` gcloud builds submit --config cloudbuild.yaml . `

You will see output from the command that ends with text like this:

Output:

Finished Step #1
ERROR
ERROR: build step 1 "gcr.io/ivil-charmer-227922klabs-gcp-49ab2930eea04/quickstart-image" failed: exit status 127
----------------------------------------------------------------------------------------------------------------------------------------------------------------
ERROR: (gcloud.builds.submit) build f3e94c28-fba4-4012-a419-48e90fca7491 completed with status "FAILURE"

Confirm that your command shell knows that the build failed:

` echo $? `

The command will reply with a non-zero value. If you had embedded this build in a script, your script would be able to act up on the build's failure.
