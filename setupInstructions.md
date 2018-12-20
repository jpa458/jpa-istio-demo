# Setting up your Demo Environment
These instructions explain how to setup your environment ready for a demo.
Assumes you have some knowledge of GCP, are comfortable with Cloud Shell, editing files from a terminal and using git.

## Configure Google Cloud platform
1. Create a new project in GCP and go to cloud shell - you can delete the project when you are done with it.

2. Run the following command and make sure your are using the correct account

  `gcloud auth list`

3. Run the following command and make sure the correct GCP project is selected

  `gcloud config list project`

4. If the project is incorrect use the following command to fix it and replace <YOUR_PROJECT_ID>

  `gcloud config set project <YOUR_PROJECT_ID>`

5. Setup the default region/zone

  `gcloud config set compute/zone europe-west1-c`

6. Enable the APIs
  ```
  gcloud services enable container.googleapis.com
  gcloud services enable cloudbuild.googleapis.com
  ```

## Setup the Kubernetes cluster
1. Create the kubernetes cluster
```
gcloud container clusters create istio-demo-cluster \
    --cluster-version=latest \
    --num-nodes 5
```

2. Setup the credentials locally so you can interact with the clusters

  ```gcloud container clusters get-credentials istio-demo-cluster```

3. Create a cluster admin role binding for your user
```
kubectl create clusterrolebinding cluster-admin-binding \
--clusterrole=cluster-admin \
--user=$(gcloud config get-value core/account)
```

4. We'll now install Istio manually by first downloading the release
(Note. Istio preinstalled in GKE is currently in Beta, we'll use our own version here)
```
curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.0.0 sh -
cd ./istio-1.0.0
```

5. Install the default Istio demo configuration into the cluster
```
kubectl apply -f install/kubernetes/istio-demo.yaml
kubectl label namespace default istio-injection=enabled
cd ..
```

## Deploy our app

1. Download our webserver app. This is a simple nodejs webapp that serves a blank colored html page.
```
git clone https://github.com/jpa458/jpa-istio-demo
cd jpa-istio-demo
```

2. Build the app to create our first 'blue' container image and push to your GCP project's container repository.

    First replace the <YOUR_PROJECT_ID> tag with the name of your GCP project in the file *cloud-build-initial.yaml*.  
    Then run the following command to execute the build

    `gcloud builds submit --config cloud-build-initial.yaml .`

3. Modify the webserver/server.js file and change the background-color style to 'yellow'

4. Rebuild the app to create our 'yellow' canary container image and push to the repository.
First replace the <YOUR_PROJECT_ID> tag with the name of your GCP project in the file cloud-build-canary.yaml
Then run the following command to execute the build
`gcloud builds submit --config cloud-build-canary.yaml .`

5. We'll now deploy our application to the cluster
```
kubectl apply -f deployment.yaml
kubectl apply -f gateway.yaml
kubectl apply -f default-routing.yaml
```

6. We now get the cluster's EXTERNAL_IP address
```
kubectl get svc istio-ingressgateway -n istio-system
```

7. Note the EXTERNAL_IP address and access the service in the browser.  With repeated calls you should see the color change between blue and yellow. The default routing will balance the requests 50/50 over both deployments

`http://<YOUR_EXTERNAL_IP_ADDRESS>/webserver`

## Demo Network Routing options
1. Now let's visualise this across 100 requests. Download the *demo.html* file to your local device. GCP Cloud Console has a feature to do this or simply clone the repo to your device.

2. Modify the IP address in demo.html to point to the cluster's external IP address.

3. Open up the local *demo.html* file in your browser. You should see 100 tiles with a 50/50 distribution of blue/yellow.

4. Run the following command - All tiles should turn blue
`kubectl apply -f routing-blue.yaml`

5. Run the following command - All tiles should turn yellow
`kubectl apply -f routing-yellow.yaml`

6. Run the following command - Tiles will be 80/20 split across blue/yellow respectively
`kubectl apply -f routing-canary.yaml`

7. You can now proceed to the *cicdPipelineSetup.md*
