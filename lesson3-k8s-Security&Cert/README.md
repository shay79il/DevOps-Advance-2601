mkdir cert && cd cert

<!-- Create Private key -->
openssl genrsa -out user1.key 2048

<!-- Create CSR  -->
openssl req -new -key user1.key -out user1.csr -subj "/CN=user1/O=group1"


<!-- Send the created CSR from the previous step and send to the CA and get the final SIGNED crt -->
openssl x509 -req -in user1.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out user1.crt -days 500


<!-- Create new user in kube config with the relevant crt -->
kubectl config set-credentials user1 --client-certificate=user1.crt --client-key=user1.key

<!-- create new context in kube config -->
kubectl config set-context user1-context --cluster=minikube --user=user1



<!-- Create role (authorization) to list pods  -->
kubectl apply -f https://raw.githubusercontent.com/avielb/k8s-experts/master/security/infrastructure/role.yaml


<!-- Bind the role (authorization) to the new user  -->
kubectl apply -f https://raw.githubusercontent.com/avielb/k8s-experts/master/security/infrastructure/role-binding.yaml


<!-- Set the new context to use  -->
kubectl config use-context user1-context

