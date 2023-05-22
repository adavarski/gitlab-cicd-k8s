# Prepare on-prem infrastructure 

Note: We will use 3 servers for this demo:

- devops (gitlab install) : 192.168.1.99
- k8s-staging: 192.168.1.100 
- k8s-production: 192.168.1.101


## Gitlab server install via ansible (devops server)

```
$ ansible-playbook -i ./inventory.ini gitlab.yml
```

### gitlab-runner setup (docker-based)

```
# curl -LJO "https://gitlab-runner-downloads.s3.amazonaws.com/latest/deb/gitlab-runner_amd64.deb"
# dpkg -i gitlab-runner_amd64.deb
# cp gitlab-runner/config.toml /etc/gitlab-runner/config.toml 
# systemctl restart gitlab-runner  
```   
## Install k8s on k8s servers and setup namespaces and GitLab credentials
  
Note: we will use k3s. k3s is 40MB binary that runs “a fully compliant production-grade Kubernetes distribution” and requires only 512MB of RAM. k3s is a great way to wrap applications that you may not want to run in a full production Cluster but would like to achieve greater uniformity in systems deployment, monitoring, and management across all development operations. 

```
### Staging Example 
$ curl -sfL https://get.k3s.io | sh -
$ sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/k8s-staging
$ sudo chmod 755 ~/.kube/k8s-staging
$ sed -i "s/127.0.0.1/192.168.1.100/"  ~/.kube/k8s-staging
$ export KUBECONFIG=~/.kube/k8s-staging
$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.1.100:6443
Metrics-server is running at https://192.168.1.100:6443/api/v1/namespaces/kube-system/services/https:metrics-server:https/proxy
CoreDNS is running at https://192.168.1.100:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

### Create k8s namespace and setup GitLab registry credentials 
$ kubectl create namespace staging

$ echo -n "root:U1AFmqZzOipO660SQgVKKrDS9qvkwsVouANA6mLXkiY=" | base64
cm9vdDpVMUFGbXFaek9pcE82NjBTUWdWS0tyRFM5cXZrd3NWb3VBTkE2bUxYa2lZPQ==

$ cat .dockerconfigjson 
{
    "auths": {
        "https://gitlab.devops.davar.com:2053":{
            "username":"root",
            "password":"U1AFmqZzOipO660SQgVKKrDS9qvkwsVouANA6mLXkiY=",
            "email": gitlab-cicd-davar@gmail.com",
            "auth":"cm9vdDpVMUFGbXFaek9pcE82NjBTUWdWS0tyRFM5cXZrd3NWb3VBTkE2bUxYa2lZPQ=="
    	}
    }
}

$ cat .dockerconfigjson | base64
ewogICAgImF1dGhzIjogewogICAgICAgICJodHRwczovL2dpdGxhYi5kZXZvcHMuZGF2YXIuY29tOjIwNTMiOnsKICAgICAgICAgICAgInVzZXJuYW1lIjoicm9vdCIsCiAgICAgICAgICAgICJwYXNzd29yZCI6IlUxQUZtcVp6T2lwTzY2MFNRZ1ZLS3JEUzlxdmt3c1ZvdUFOQTZtTFhraVk9IiwKICAgICAgICAgICAgImVtYWlsIjoiYW5hc3Rhcy5kYXZhcnNraUBnbWFpbC5jb20iLAogICAgICAgICAgICAiYXV0aCI6ImNtOXZkRHBWTVVGR2JYRmFlazlwY0U4Mk5qQlRVV2RXUzB0eVJGTTVjWFpyZDNOV2IzVkJUa0UyYlV4WWEybFpQUT09IgogICAgCX0KICAgIH0KfQo=

$ cat registry-credentials.yml 
apiVersion: v1
kind: Secret
metadata:
  name: registry-credentials
  namespace: staging
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: ewogICAgImF1dGhzIjogewogICAgICAgICJodHRwczovL2dpdGxhYi5kZXZvcHMuZGF2YXIuY29tOjIwNTMiOnsKICAgICAgICAgICAgInVzZXJuYW1lIjoicm9vdCIsCiAgICAgICAgICAgICJwYXNzd29yZCI6IlUxQUZtcVp6T2lwTzY2MFNRZ1ZLS3JEUzlxdmt3c1ZvdUFOQTZtTFhraVk9IiwKICAgICAgICAgICAgImVtYWlsIjoiYW5hc3Rhcy5kYXZhcnNraUBnbWFpbC5jb20iLAogICAgICAgICAgICAiYXV0aCI6ImNtOXZkRHBWTVVGR2JYRmFlazlwY0U4Mk5qQlRVV2RXUzB0eVJGTTVjWFpyZDNOV2IzVkJUa0UyYlV4WWEybFpQUT09IgogICAgCX0KICAgIH0KfQo=

$ kubectl create -f registry-credentials.yml 
secret/registry-credentials created

```
## Create Gitlab project and push endurosat-cicd folder to GitLab repo, create Gitlab CI/CD variables and .gitlab-ci.yml pipeline file:

Screenshots: 

Repo:

<img src="./pictures/endurosat-gitlab-repo.png?raw=true" width="900">

Variables(K8S_STAGING=~/.kube/k8s-staging):

<img src="./pictures/endurosat-gitlab-pipilene-variables.png?raw=true" width="900">

Pipeline Run:

<img src="./pictures/endurosat-gitlab-pipeline-summary.png?raw=true" width="900">

Pipeline state -> build docker image:

<img src="./pictures/endurosat-state-build.png?raw=true" width="900">

Pipeline state -> test: 

<img src="./pictures/endurosat-state-test.png?raw=true" width="900">

Pipeline state -> push docker image to GitLab repo:

<img src="./pictures/endurosat-state-push.pngg?raw=true" width="900">

Pipeline state -> staging deploy:

<img src="./pictures/endurosat-state-staging-deploy.png?raw=true" width="900">

Pipeline state -> production deploy:

<img src="./pictures/endurosat-state-production-deploy.png?raw=true" width="900">


### Check deplpyment (staging k8s example):

```

$ kubectl get deployment -n staging
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
endurosat-cicd   1/1     1            1           38h

$ kubectl get po -n staging
NAME                              READY   STATUS    RESTARTS   AGE
endurosat-cicd-6f8fd6596b-jw2js   1/1     Running   0          38h

```









