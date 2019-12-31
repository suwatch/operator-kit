$ mkdir C:\Users\suwatch\go\bin
$ mkdir C:\Users\suwatch\go\src

REM clone repo
$ cd /C/Users/suwatch/go/src
$ git clone https://github.com/rook/operator-kit
$ cd /C/Users/suwatch/go/src/operator-kit

REM install go-dep at /c/Users/suwatch/go/bin
$ export GOBIN=/c/Users/suwatch/go/bin
$ curl https://raw.githubusercontent.com/golang/dep/master/install.sh | sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  5230  100  5230    0     0  11545      0 --:--:-- --:--:-- --:--:-- 11545
ARCH = amd64
OS = windows
Will install into /c/Users/suwatch/go/bin
Fetching https://github.com/golang/dep/releases/latest..
Release Tag = v0.5.4
Fetching https://github.com/golang/dep/releases/tag/v0.5.4..
Fetching https://github.com/golang/dep/releases/download/v0.5.4/dep-windows-amd64.exe..
Setting executable permissions.
Moving executable to /c/Users/suwatch/go/bin/dep.exe

REM download dependency to C:\Users\suwatch\go\pkg\dep\sources 
REM this may take minutes
$ dep ensure

$ cd /C/Users/suwatch/go/src/operator-kit/sample-operator

REM this will build for linux output sample-operator
$ CGO_ENABLED=0 GOOS=linux go build

REM build docker
$ docker build -t sample-operator:0.1 .
$ docker images sample-operator:0.1

REM create ServiceAccount (sample-operator) and grant RBAC (clusterrole and clusterrolebinding)
REM create deployment with ServiceAccount (sample-operator) and Container (sample-operator) with image (sample-operator:0.1)
$ kubectl create -f /C/Users/suwatch/go/src/operator-kit/sample-operator/sample-operator.yaml
clusterrole.rbac.authorization.k8s.io/sample-operator created
serviceaccount/sample-operator created
clusterrolebinding.rbac.authorization.k8s.io/sample-operator created
deployment.apps/sample-operator created

REM get all with label app=sample-operator
$ kubectl get all -l app=sample-operator
NAME                                   READY   STATUS    RESTARTS   AGE
pod/sample-operator-68fd7c6598-svrbf   1/1     Running   0          2m10s
NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/sample-operator   1/1     1            1           2m10s
NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/sample-operator-68fd7c6598   1         1         1       2m10s

REM once pod is running, you can list all crd
$ kubectl get crd
NAME                   CREATED AT
samples.myproject.io   2019-12-31T20:50:02Z

REM once pod is running, you can see the log from container
$ kubectl logs -l app=sample-operator
Getting kubernetes context
Registering the sample resource
Watching the sample resource


REM create a sample resource
$ kubectl create -f /C/Users/suwatch/go/src/operator-kit/sample-operator/sample-resource.yaml
sample.myproject.io/mysample created

REM check the log, see new resource added
$ kubectl logs -l app=sample-operator
Getting kubernetes context
Registering the sample resource
Watching the sample resource
Added Sample 'mysample' with Hello=world

REM list resource with sample kind
$ kubectl get samples.myproject.io
NAME       AGE
mysample   2m46s

REM make change ( hello: world to hello: suwatch)
$ kubectl apply -f /C/Users/suwatch/go/src/operator-kit/sample-operator/sample-resource-update.yaml
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
sample.myproject.io/mysample configured

REM check the log, see propert change
$ kubectl logs -l app=sample-operator
Getting kubernetes context
Registering the sample resource
Watching the sample resource
Added Sample 'mysample' with Hello=world
Updated sample 'mysample' from world to suwatch

REM cleanup 
$ kubectl delete -f sample-resource.yaml
$ kubectl delete -f sample-operator.yaml
$ kubectl delete crd samples.myproject.io


