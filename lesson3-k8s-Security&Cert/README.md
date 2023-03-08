

# Process to create new certificate for a USER 
## To be authenticated and recognized by the K8S API_SERVER

```bash
mkdir cert && cd cert

# Create Private key 
openssl genrsa -out user1.key 2048

# Create CSR
openssl req -new -key user1.key -out user1.csr -subj "/CN=user1/O=group1"

# Send the created CSR from the previous step and send to the CA and get the final SIGNED crt
openssl x509 -req -in user1.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out user1.crt -days 500

# Combine the `crt` and the `key` to one `pfx` file to be uploaded to CHROME
openssl pkcs12 -export -in user1.crt -inkey user1.key -out user1.pfx

# Create new user in kube config with the relevant crt
kubectl config set-credentials user1 --client-certificate=user1.crt --client-key=user1.key

# create new context in kube config 
kubectl config set-context user1-context --cluster=minikube --user=user1

# Create role (authorization) to list pods 
kubectl apply -f https://raw.githubusercontent.com/avielb/k8s-experts/master/security/infrastructure/role.yaml


# Bind the role (authorization) to the new user  
kubectl apply -f https://raw.githubusercontent.com/avielb/k8s-experts/master/security/infrastructure/role-binding.yaml


# Set the new context to use
kubectl config use-context user1-context
```

# Process to create new SELF SIGNED certificate 

## The SELF SIGNED certificate will be used by K8S web-server which use a secret 
## With both the private key and the `crt` file (with the relevent DNS name) for the HTTPS session 
## The web browser (e.g. CHROME) will be uploaded with the CA cert in order to authenticate the web server that runs on K8S

```bash

# Create Private key 
openssl genrsa -out myk8s.key 2048

# Create CSR
openssl req -new -nodes -key myk8s.key -out myk8s.csr

# Add DNS name
echo "subjectAltName=DNS:myk8s.com" >> extfile.cnf

# Send the created CSR from the previous step and send to the CA and get the final SIGNED crt
openssl x509 -req -in myk8s.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out myk8s.crt -extfile extfile.cnf -days 365

# Create a secret to be used by the web-server ingress resource
kubectl create secret tls tls-secret --key myk8s.key --cert myk8s.crt --certificate-authority ~/.minikube/ca.crt  -o yaml --dry-run=client > tls-secret.yaml
```