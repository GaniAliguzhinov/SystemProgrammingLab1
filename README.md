
# System Programming Lab (17.02.2021)

## Initialize the Console

* Open Google Console in a private tab (Best to use Google Chrome).
* Log in using generated Username/Password (Don't save, Confirm, Agree to License)
* Activate Google Cloud Shell (GCS).

Click on + to provision a new shell if current shell is not starting.

Google Cloud Shell allows creating files on the instance, as well as access to instance management commands with `gcloud` and `gsutil`.

On first usage, click authorize to authorize shell.

## Set Parameters

Copy PROJECT_ID from "GCP Project ID" (same tab as credentials and "End Lab" button).

Come up with a name for the bucket, e.g., john-smith-startup-bucket

* Set variables in the GCS:
```sh
export PROJECT_ID=[PROJECT_ID]
export BUCKET_NAME=[BUCKET_NAME]
```

In Home->Dashboard->Resources->Compute Engine we can view a list of instances.
We should have just one named "lab-monitor". To view details, click Settings(Dots)->View monitoring->Details tab.
There, we can connect to serial console to view instance logs.

* Let GCS know which project we are working on:
```sh
gcloud compute project-info describe --project $PROJECT_ID # Check default-zone and default-region, should be us-central1-a
gcloud config set project $PROJECT_ID
```

* Set region and zone of the instance (where should instance be located):
```sh
gcloud compute project-info add-metadata \
   --metadata=google-compute-default-region=us-central1,google-compute-default-zone=us-central1-a
gcloud init # 1 1 5
```

## Add a remote startup script

* Create Bucket:
```sh
gsutil mb -b on -l us-east2 gs://$BUCKET_NAME
```

* Create startup script inside GCS:
```sh
cat > install-web.sh <<EOF
#!/bin/bash
apt-get update
apt-get -y install apache2
EOF
cat install-web.sh
```

* Copy script to the bucket:
```sh
gsutil cp install-web.sh gs://$BUCKET_NAME/
```

* Set the file on the bucket as the startup script for the instance:
```sh
gcloud compute instances add-metadata lab-monitor --metadata startup-script-url=gs://$BUCKET_NAME/install-web.sh
```

## Set firewall rules

* Create a firewall rule for the project. It should have a name, apply for inbound traffic (INGRESS), 
should apply to tcp traffic on port 80 (HTTP), and should apply to traffic from all sources (mask is 0.0.0.0/0).
Finally, Google Cloud manages firewall rules with network tags, so we should also specify a new network tag 
that our rule should apply to:
```sh
gcloud compute --project=$PROJECT_ID firewall-rules create http-rule --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
```

* Now, we need to add current instance to the network group:
```sh
gcloud compute instances add-tags lab-monitor \
    --zone us-central1-a \
    --tags http-server
```

## Restart Instance

* Go to Dashboard->Compute Engine, check the instance, and click stop.

* Wait for instance to stop, then press Start/Resume.

While waiting for instance to start, go to the Details tab of the View Monitoring of the instance. 
There, click on Serial Port 1 (console), and keep refreshing until you see something like:

`Feb 15 17:35:21 lab-monitor systemd[1]: Startup finished in 1.803s (kernel) + 43.280s (userspace) = 45.084s.`

Everything should work now, which you can check by checking progress on the page with the "End Lab" button.

Finally, make a screenshot (On Windows, PrtScr, then open Paint and Ctrl+V) of the tab so we can see that you got 100/100.

Go to Google Forms link, enter your Name, Student ID, Answer some questions, and upload the screenshot.
