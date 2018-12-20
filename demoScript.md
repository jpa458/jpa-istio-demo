

19. You can change the routing rules by running the following commands and verifying the effect in your browser. Default routing will balance requests 50/50
kubectl apply -f default-routing.yaml
kubectl apply -f static-routing-blue.yaml
kubectl apply -f static-routing-yellow.yaml

kubectl get services
kubectl get pods

export GATEWAY_URL=<your gateway IP>
