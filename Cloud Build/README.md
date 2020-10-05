

Enable APIs and Services

* Cloud Build

* Container Registry


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
