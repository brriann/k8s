# AuthN walkthrough
- https://www.edx.org/learn/kubernetes/the-linux-foundation-introduction-to-kubernetes

sudo useradd -s /bin/bash bob
sudo passwd bob

Create a private key for the new user bob with the openssl tool, then create a certificate signing request for bob with the same openssl tool:

openssl genrsa -out bob.key 2048

openssl req -new -key bob.key \
  -out bob.csr -subj "/CN=bob/O=learner"

  Create a YAML definition manifest for a certificate signing request object, and save it with a blank value for the request field: 

  vim signing-request.yaml

  apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: bob-csr
spec:
  groups:
  - system:authenticated
  request: <assign encoded value from next cat command>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - digital signature
  - key encipherment
  - client auth

View the certificate, encode it in base64, and assign it to the request field in the signing-request.yaml file:

cat bob.csr | base64 | tr -d '\n','%'

vim signing-request.yaml

apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: bob-csr
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUd...1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - digital signature
  - key encipherment
  - client auth

Create the certificate signing request object, then list the certificate signing request objects. It shows a pending state:

kubectl create -f signing-request.yaml

kubectl get csr

Approve the certificate signing request object, then list the certificate signing request objects again. It shows both approved and issued states:

kubectl certificate approve bob-csr

kubectl get csr

Extract the approved certificate from the certificate signing request, decode it with base64 and save it as a certificate file. Then view the certificate in the newly created certificate file:

kubectl get csr bob-csr \
  -o jsonpath='{.status.certificate}' | \
  base64 -d > bob.crt

cat bob.crt

-----BEGIN CERTIFICATE-----
MIIDGzCCA...
...
...NOZRRZBVunTjK7A==
-----END CERTIFICATE-----

Configure the kubectl client's configuration manifest with user bob's credentials by assigning his key and certificate: 

kubectl config set-credentials bob \
  --client-certificate=bob.crt --client-key=bob.key

User "bob" set.

Create a new context entry in the kubectl client's configuration manifest for user bob, associated with the lfs158 namespace in the minikube cluster:

kubectl config set-context bob-context \
  --cluster=minikube --namespace=lfs158 --user=bob

Context "bob-context" created.

View the contents of the kubectl client's configuration manifest again, observing the new context entry bob-context, and the new user entry bob (the output is redacted for readability):

kubectl config view

While in the default minikube context, create a new deployment in the lfs158 namespace:

kubectl -n lfs158 create deployment nginx --image=nginx:alpine

deployment.apps/nginx created

From the new context bob-context try to list pods. The attempt fails because user bob has no permissions configured for the bob-context:

kubectl --context=bob-context get pods

The following steps will assign a limited set of permissions to user bob in the bob-context. 

Create a YAML configuration manifest for a pod-reader Role object, which allows only get, watch, list actions/verbs in the lfs158 namespace against pod resources. Then create the role object and list it from the default minikube context, but from the lfs158 namespace:

vim role.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: lfs158
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

kubectl create -f role.yaml

role.rbac.authorization.k8s.io/pod-reader created

kubectl -n lfs158 get roles

Create a YAML configuration manifest for a rolebinding object, which assigns the permissions of the pod-reader Role to user bob. Then create the rolebinding object and list it from the default minikube context, but from the lfs158 namespace:

vim rolebinding.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-read-access
  namespace: lfs158
subjects:
- kind: User
  name: bob
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

kubectl create -f rolebinding.yaml 

rolebinding.rbac.authorization.k8s.io/pod-read-access created

kubectl -n lfs158 get rolebindings

NAME              ROLE              AGE
pod-read-access   Role/pod-reader   28s

Now that we have assigned permissions to bob, we can successfully list pods from the new context bob-context.

kubectl --context=bob-context get pods
