# Kubernetes Course

### Docker

Install **_[docker for mac](https://docs.docker.com/docker-for-mac/install/)._**  
Follow **_[docker guide](https://docs.docker.com/docker-for-mac/#advanced)_** to increase resource limit(need to set memory to 8GB+).

```
$ brew install Docker

$ docker version
Client: Docker Engine - Community
 Version:           19.03.2
 API version:       1.40
 Go version:        go1.12.8
 Git commit:        6a30dfc
 Built:             Thu Aug 29 05:26:49 2019
 OS/Arch:           darwin/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.2
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.8
  Git commit:       6a30dfc
  Built:            Thu Aug 29 05:32:21 2019
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v1.2.6
  GitCommit:        894b81a4b802e4eb2a91d1ce216b8817763c29fb
 runc:
  Version:          1.0.0-rc8
  GitCommit:        425e105d5a03fabd737a126ad93d62a9eeede87f
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```

### Go
Install Go download from **_[golang.org](https://golang.org/doc/install)_**
```
$ brew install go

$ go version
go version go1.12.9 darwin/amd64

$ go env
GOARCH="amd64"
GOBIN=""
GOCACHE="/Users/has3ong/Library/Caches/go-build"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOOS="darwin"
GOPATH="/Users/has3ong/go"
GOPROXY=""
GORACE=""
GOROOT="/usr/local/Cellar/go/1.12.9/libexec"
GOTMPDIR=""
GOTOOLDIR="/usr/local/Cellar/go/1.12.9/libexec/pkg/tool/darwin_amd64"
GCCGO="gccgo"
CC="clang"
CXX="clang++"
CGO_ENABLED="1"
GOMOD=""
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/var/folders/yq/4wxz887d6sb1xh2zqf9l4_100000gn/T/go-build884843357=/tmp/go-build -gno-record-gcc-switches -fno-common"
```  

### Kubectl

```
### Install with Homebrew on macOS
$ brew install kubernetes-cli

### download kubectl 1.16.0
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.16.0/bin/darwin/amd64/kubectl

$ chmod -x ./kubectl
    
$ mv ./kubectl /usr/local/bin/kubectl

### if kubectl isn't executable
$ chmod 755 /usr/local/bin/kubectl
```
    
### KIND

Refer to _**[KIND](https://github.com/kubernetes-sigs/kind)**_ for more details

```
$ GO111MODULE="on" go get sigs.k8s.io/kind@v0.5.1

$ export PATH="$PATH:$(go env GOPATH)/bin"
```

### Aliases

```
$ alias k='kubectl'
    
$ alias chrome="/Applications/Google\\ \\Chrome.app/Contents/MacOS/Google\\ \\Chrome"
```

### Useful Tools

_kubectl_ command shell _**[auto-completion](https://kubernetes.io/docs/tasks/tools/install-kubectl/?source=#enabling-shell-autocompletion)**_
  
```
### for oh-my-zsh
$ plugins=(... kubectl)
```

* k8s context/namespace changer _**[kubectx/kubens](https://github.com/ahmetb/kubectx)**_  
* Awesome k8s shell prompt _**[kube ps1](https://github.com/jonmosco/kube-ps1)**_  
* Very cool k8s CLI manage tool _**[k9s](https://k9ss.io/?fbclid=IwAR0MQO9yBF5iKpJlDkuSNtrWGy72zK81I-j071lrKQsV1DLhloOMknOLd64)**_  
* Multiple pods log tool for k8s _**[stern](https://github.com/wercker/stern)**_

## Practice

### Create k8s multi-node cluster with KIND
Will create 1 master and 2 worker nodes.  

```
$ kind create cluster --name kind-m --config kind-test-config.yaml
```

### Change k8s config  
Add configs from _'.kube/kind-config-kind-m'_ to _'.kube/config'_ file
or:

```
$ export KUBECONFIG="$(kind get kubeconfig-path --name="kind-m")"
```


### Verify k8s cluster

```
$ kubectl cluster-info
$ kubectl get nodes
$ kubectl get pods --all-namespaces
```

### Install metrics-server
Refer to _**[metrics-server](https://github.com/kubernetes-incubator/metrics-server)**_ for more information

```
$ kubectl apply -f ./metrics-server
```

### Install Ingress-nginx
Installation _**[Guide](https://kubernetes.github.io/ingress-nginx/deploy/)**_

```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml

# create service with nodeport 32080, 32433
$ kubectl apply -f ./ingress-nginx/
```

### Deploy MariaDB(MySQL) for Wordpress - Stateful Sets
Deploy MariaDB

```    
$ kubectl create namespace wordpress
$ kubectl apply -f ./mysql

# Check Stateful Sets
$ kubectl get sts -n wordpress
$ kubectl describe sts mysql -n wordpress
```

### Deploy Wordpress - Deployment

Deploy WordPress

```
# get you local ip
$ IP=$(ifconfig | grep "inet " | grep -Fv 127.0.0.1 | awk '{print $2}')

# replace ingress host with your IP
$ sed -i.bak "s/host.*/host: blog."$IP".nip.io/g" ./wordpress/ingress.yaml
    
$ kubectl apply -f ./wordpress

# edit deployment
$ kubectl edit deploy wordpress

# port forward to check if wordpress is running correctly
$ WPOD=$(kubectl get -n wordpress pod -l app=wordpress -o jsonpath="{.items[0].metadata.name}")

$ kubectl port-forward pod/$WPOD 8090:80 -n wordpress
```

Scale in/out Deployments

```
$ kubectl scale deploy wordpress --replicas=3 -n wordpress
$ kubectl scale deploy wordpress --replicas=1 -n wordpress

# check services for wordpress
$ kubectl get svc -n wordpress

# check ingress
$ kubectl get ingress -n wordpress

# other commands
$ k explain deployment --recursive
$ k explain svc --recursive
$ k get pods -o wide --sort-by="{.spec.nodeName}"
```

### Expose Ingress
Intall socat to expose nodeport on local 80 port

```
$ docker run -d --name kind-proxy-80 \
--publish 80:80 \
--link kind-m-control-plane:target \
alpine/socat \
tcp-listen:80,fork,reuseaddr tcp-connect:target:32080
```

### Test

Test with your own DNS

```
$ chrome http://blog.$IP.nip.io
```

### Delete resources and cluster

```
$ kubectl --namespace=wordpress delete --all
$ kubectl delete --namespace wordpress

$ kind delete cluster --name kind-m

# remove socat
$ docker rm -f kind-proxy-80
```

