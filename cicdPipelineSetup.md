# Playing with CI/CD Pipelines
Assumes you have just completed the setupInstructions.md steps and can visualise the 80/20 canary release distribution of blue/yellow tiles in demo.html page.

## Run the CI/CD build script locally in cloud shell
1. Change the background-color style in the webserver/server.js file to another colour eg 'red'

2. We now need to allow Cloud Build to deploy to the GKE cluster by adding a GKE IAM role:
  * In GCP Console, visit the IAM menu.
  * From the list of service accounts, click the Roles drop-down menu beside the
  * Cloud Build <YOUR-PROJECT-ID>@cloudbuild.gserviceaccount.com service account.
  * Click Kubernetes Engine, then click Kubernetes Engine Admin.
  * Click Save.
  * You can also do this via the command line:    
    `gcloud projects add-iam-policy-binding <PROJECT_ID> \
        --member serviceAccount:<PROJECT_NUMBER>@cloudbuild.gserviceaccount.com --role roles/container.admin`

2. Verify the CI/CD build script works when run locally. First update the <YOUR_PROJECT_ID> tags in the file.

  `gcloud builds submit --config cloud-build-cicd.yaml .`

3. This will build the image, push it to the container respository and trigger a rolling deployment.

4. Load up the demo.html file and watch the existing canary release (yellow?) gradually rollover to the new red version.

## Trigger automated deployments on source code commits

1. Fork the github repo into your own github account. (You could also use Bitbucket or GCP Cloud Repository but we'll go with github here)

2. Create a GCP build trigger on your forked github repo:
  * GCP Console->Cloud Build->Triggers->Create and follow the steps to hook up the trigger with your repository
  * Point the build script to the cloud-build-cicd.yaml file

3. Change the background-color style in webserver/server.js to another color eg. 'white'

4. Commit your change and push it to the repository
Monitor the demo.html file - the canaries should gradually roll over to white
You also can view the build in progress and follow the log status in the GCP console Cloud Build history.
