`OVERVIEW OF PROJECT`    

This project automates canary deployment of various versions of a flask app with istio & flagger(https://flagger.app/) while circleci is used for continuous integration. A Dockerfile is used to build an image and deploy to a container registry (Dockerhub) then sets the new version on the cluster. It deploys only 50% of the new traffic for one minute and only deploys to all the traffic if there are no 5xx errors in the canary deployment for a minute.

`STEPS TO REPRODUCE`:

- Create a cluster 

- Go to the correct configuration folder

- Connect to the cluster 

- Grant cluster administrator (admin) permissions to the current user
kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$

- Download and install Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.11.3
export PATH=$PWD/bin:$PATH
istioctl install --set profile=demo -y

- Create namespace 
kubectl create namespace canary-test

- Add a namespace label to instruct Istio to automatically inject Envoy sidecar proxies when you deploy your application later:
kubectl label namespace canary-test istio-injection=enabled

- Deploy your application
kubectl apply -f ../application.yaml

kubectl get pods
kubectl get deploy
kubectl get svc
kubectl get serviceaccount

- Expose your application
kubectl apply -f ../gateway.yaml

kubectl get gateway
kubectl get vs

- Check configurations
istioctl analyze -n canary-test

- Access the application from the browser
Get external ip from:
kubectl get svc istio-ingressgateway -n istio-system

- Install prometheus
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.11/samples/addons/prometheus.yaml

- Install grafana(optional)
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.11/samples/addons/grafana.yaml

- Install flagger
kubectl apply -k github.com/fluxcd/flagger//kustomize/istio if this doesn't work you can download the repo on (https://github.com/fluxcd/flagger) and run: kubectl apply -k <path-to-file>/flagger-main/kustomize/istio

- Deploy canary
kubectl apply -f ../canary.yaml

- Wait until canary is initialized 
kubectl get pods -n istio-system 
kubectl logs -f <flagger-remaining-pod-name> -n istio-system

- Access the application from the browser

- Trigger a canary deployment
## Manual:
kubectl set image deployment/flask-app flask-app=lexmill99/canary-flask-app:v2 -n canary-test

## Automatic:
To setup circleci with your github repo, follow the instructions here: https://circleci.com/blog/setting-up-continuous-integration-with-github/

then make a push to github and circleci will trigger the deployment

- Wait until canary is deployed with the correct weight 
kubectl logs -f <flagger-remaining-pod-name> -n istio-system


`Video showing proof-of-concept shift from v2 to v1`:
https://drive.google.com/file/d/1Sj_sohwmdUQPZ2iHH4By4-CgxpaFBPrM/view?usp=sharing

Circle/CI is used for the CI part and the configuration is defined in .circleci/config.yml


`Screenshot showing CIRCLE/CI pipeline`:
https://drive.google.com/file/d/1krUILjxwC-dqlN02yAbJJDXdlcn9RW-0/view?usp=sharing


`Contributing`
- Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

- Please make sure to update tests as appropriate.

- Last updated: 2021.10.05.
