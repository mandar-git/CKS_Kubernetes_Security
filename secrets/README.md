
# Create secret from a string literal(s)

```
kubectl create secret generic my-db-secret --from-literal=DB_ROOT_USER=db_admin_root --from-literal=DB_ROOT_PWD=secret-db-pass
```

`secret/my-db-secret created`


```kubectl get secret my-db-secret```
`
NAME                  TYPE                                  DATA   AGE
my-db-secret          Opaque                                2      6s
`

```
kubectl get secret my-db-secret -o yaml
```
`
apiVersion: v1
data:
  DB_ROOT_PWD: c2VjcmV0LWRiLXBhc3M=
  DB_ROOT_USER: ZGJfYWRtaW5fcm9vdA==
kind: Secret
metadata:
  creationTimestamp: "2021-12-12T14:51:08Z"
  name: my-db-secret
  namespace: default
  resourceVersion: "2281"
  uid: 72710505-3a2b-400a-8c62-42661728d625
type: Opaque
`

```
kubectl describe secret my-db-secret
```
```
Name:         my-db-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
DB_ROOT_PWD:   14 bytes
DB_ROOT_USER:  13 bytes
```
  
  
# Create secret without encoding as base64 , only to be used for basic authentication to store username and password fields
  
```
kubectl create secret generic test-secret --dry-run=client -o yaml > secret.yaml
```
  
```
cat secret.yaml
```

```
apiVersion: v1
kind: Secret
metadata:
  creationTimestamp: null
  name: test-secret
```  
Edit the yaml manifest to use StringData and add username and password. As the type is basic-auth, the data fields must only contain  username and password

`vi secret.yaml`

```  
apiVersion: v1
kind: Secret
metadata:
  name: test-secret
type: kubernetes.io/basic-auth
stringData:
  username: db-admin
  password: my-secret-d8-p@ssw0rd
```
kubectl create -f secret.yaml
```
secret/test-secret created
```
`kubectl get secret test-secret`
```
NAME          TYPE                       DATA   AGE
test-secret   kubernetes.io/basic-auth   2      17s
```

`kubectl get secret test-secret -o yaml`
```
apiVersion: v1
data:
  password: bXktc2VjcmV0LWQ4LXBAc3N3MHJk
  username: ZGItYWRtaW4=
kind: Secret
metadata:
  creationTimestamp: "2021-12-12T16:06:48Z"
  name: test-secret
  namespace: default
  resourceVersion: "7775"
  uid: 2bd2e221-dd56-4581-959e-458f72796a6a
type: kubernetes.io/basic-auth
```
 
# Create secret from file(s)
```
echo -n 'test-user' > user
echo -n 'secret-p@ssw0rd' > passwd
```
```
kubectl create secret generic file-secret --from-file=user --from-file=passwd
```
`secret/file-secret created`
```
kubectl get secret file-secret
```

```
NAME          TYPE     DATA   AGE
file-secret   Opaque   2      18s
```

```
kubectl get secret file-secret -o yaml
apiVersion: v1
data:
  pwd.txt: c2VjcmV0LXBAc3N3MHJk
  user.txt: dGVzdC11c2Vy
kind: Secret
metadata:
  creationTimestamp: "2021-12-12T15:57:25Z"
  name: file-secret
  namespace: default
  resourceVersion: "7093"
  uid: 07ecde6a-6d5c-4fc1-be2e-d04cfd0dd599
type: Opaque
```

# Retrieve secret data fields (in base64 encoded)
  
 ```
kubectl get secret file-secret -o jsonpath='{.data
```
`{"pwd.txt":"c2VjcmV0LXBAc3N3MHJk","user.txt":"dGVzdC11c2Vy"}`


# Create a secret to be used for TLS


```
openssl req -x509 -new -nodes -keyout mytls.key -out mytls.crt -subj "/CN=mydomain.test"
```

```
Generating a RSA private key
..............+++++
.....................................+++++
writing new private key to 'mytls.key'
-----
```


`ls`
```
mytls.crt  mytls.key
```

```
kubectl create secret tls my-tls-secret --cert=mytls.crt --key=mytls.key -n default
```
`secret/my-tls-secret created`
```
kubectl get secret
```
```
NAME                  TYPE                                  DATA   AGE
default-token-8tw98   kubernetes.io/service-account-token   3      20m
my-tls-secret         kubernetes.io/tls                     2      4s
```
```
kubectl describe secret my-tls-secret
```

```
Name:         my-tls-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  kubernetes.io/tls

Data
====
tls.crt:  1123 bytes
tls.key:  1704 bytes
``` 
  
  
