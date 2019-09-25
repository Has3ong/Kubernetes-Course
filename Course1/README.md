# Kubernetes Course 1

## Running Wordpress on Kubernetes

Minikube Start

```
$ minikube start
⚠️  minikube 1.4.0 is available! Download it: https://github.com/kubernetes/minikube/releases/tag/v/1.4.0
💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'
😄  minikube v1.3.1 on Darwin 10.14.6
💡  Tip: Use 'minikube start -p <name>' to create a new cluster, or 'minikube delete' to delete this one.
🔄  Starting existing virtualbox VM for "minikube" ...
⌛  Waiting for the host to be provisioned ...
🐳  Preparing Kubernetes v1.15.2 on Docker 18.09.8 ...
🔄  Relaunching Kubernetes using kubeadm ...
⌛  Waiting for: apiserver proxy etcd scheduler controller dns
🏄  Done! kubectl is now configured to use "minikube"
```

```
$ kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   20d   v1.15.2
```

#### Create Wordpress secret, delpoyment
```
$ kubectl create -f wordpress/wordpress-secrets.yml
secret/wordpress-secrets created

$ kubectl create -f wordpress/wordpress-single-deployment-no-volumes.yml
deployment.extensions/wordpress-deployment created

$ kubectl get pods
NAME                                    READY   STATUS              RESTARTS   AGE
nginx-8sjcn                             1/1     Running             1          16d
nginx-bffqg                             1/1     Running             1          16d
nginx-wnwm8                             1/1     Running             1          16d
wordpress-deployment-58cd589c6c-xvjnk   0/2     ContainerCreating   0          3s
```

#### Describe pod of Wordpress
```
$ kubectl describe pod wordpress
Name:           wordpress-deployment-58cd589c6c-xvjnk
Namespace:      default
Priority:       0
Node:           minikube/10.0.2.15
Start Time:     Wed, 25 Sep 2019 23:22:30 +0900
Labels:         app=wordpress
                pod-template-hash=58cd589c6c
Annotations:    <none>
Status:         Pending
IP:
IPs:            <none>
Controlled By:  ReplicaSet/wordpress-deployment-58cd589c6c
Containers:
  wordpress:
    Container ID:
    Image:          wordpress:4-php7.0
    Image ID:
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Environment:
      WORDPRESS_DB_PASSWORD:  <set to the key 'db-password' in secret 'wordpress-secrets'>  Optional: false
      WORDPRESS_DB_HOST:      127.0.0.1
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-8w67q (ro)
  mysql:
    Container ID:
    Image:          mysql:5.7
    Image ID:
    Port:           3306/TCP
    Host Port:      0/TCP
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Environment:
      MYSQL_ROOT_PASSWORD:  <set to the key 'db-password' in secret 'wordpress-secrets'>  Optional: false
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-8w67q (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  default-token-8w67q:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-8w67q
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  62s   default-scheduler  Successfully assigned default/wordpress-deployment-58cd589c6c-xvjnk to minikube
  Normal  Pulling    61s   kubelet, minikube  Pulling image "wordpress:4-php7.0"
  Normal  Pulled     23s   kubelet, minikube  Successfully pulled image "wordpress:4-php7.0"
  Normal  Created    23s   kubelet, minikube  Created container wordpress
  Normal  Started    23s   kubelet, minikube  Started container wordpress
  Normal  Pulling    23s   kubelet, minikube  Pulling image "mysql:5.7"

```

#### Create Wordpress Service
```
$ kubectl create -f wordpress/wordpress-service.yml
service/wordpress-service created

$ minikube service wordpress-service --url
http://192.168.99.100:31001
```

#### 192.168.99.100:31001

<img width=500 src="https://user-images.githubusercontent.com/44635266/65610597-8baa9900-dfec-11e9-9e9f-124db8c6aaab.png">

#### Registry

<img width=500 src="https://user-images.githubusercontent.com/44635266/65610598-8c432f80-dfec-11e9-8f9b-0048d6e0fa80.png">

#### Login

<img width=500 src="https://user-images.githubusercontent.com/44635266/65610595-8b120280-dfec-11e9-8937-a5da4215769f.png">

#### My Blog

<img width=500 src="https://user-images.githubusercontent.com/44635266/65610596-8baa9900-dfec-11e9-8727-716bd0192ef5.png">


## Web UI in kops


# Setting up the dashboard

## Start dashboard

Create dashboard:
```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

## Create user

Create sample user 

```
$ kubectl create -f sample-user.yaml
```

## Get login token:
```
$ kubectl -n kube-system get secret | grep admin-user

admin-user-token-mptfz                           kubernetes.io/service-account-token   3      14s

$ kubectl -n kube-system describe secret admin-user-token-<id displayed by previous command>


Name:         admin-user-token-mptfz
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: 95294dd8-6830-4c95-900d-00a3814787d1

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLW1wdGZ6Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI5NTI5NGRkOC02ODMwLTRjOTUtOTAwZC0wMGEzODE0Nzg3ZDEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.VQkMllsEZXuyvpGIk_Pq6-86tkDuWbB3oaP09mIjqbP-WYllxvc2bd9lcdfufD8wUioZy6qOprNN9wM9Y3zoONEhfguGe9qWXblfpf1LqjIDdVZcbOQt0b1D-fm_Ok136LYM9KwXxAeVtsOJxMvJwcpSf1LMhkLZVJROmPDgsNEX9vc61xr9kSH_Ejhp8siUTnDcNgx_2-394kMs7d76pl8PX5KRx_mUvQ_IvGTP4KE51Si6bxd7-3mcnRj5I2h_b1Z9_0wtTkvxN29PqQEuAiOa3Y6_nJ6UXBxspEDYogiw4eS3M1xTBb1rqSGtMXuWzrrb86izDjcCqFY7Rr56RQ
```

## Login to dashboard
```
$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/has3ong/.minikube/ca.crt
    server: https://192.168.99.100:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /Users/has3ong/.minikube/client.crt
    client-key: /Users/has3ong/.minikube/client.key

$ minikube dashboard --url
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
http://127.0.0.1:53368/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/
```

Login: admin
Password: the password that is listed in ~/.kube/config (open file in editor and look for "password: ..."

Choose for login token and enter the login token from the previous step

### Dashboard

<img width="500" src="https://user-images.githubusercontent.com/44635266/65613211-9404d300-dff0-11e9-8d45-002e528cb228.png">
