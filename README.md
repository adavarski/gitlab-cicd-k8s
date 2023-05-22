We will use 3 servers in this demo:

devops (gitlab install) : 192.168.1.99
k8s-staging: 192.168.1.100 
k8s-production: 192.168.1.101


## Gitlab server install via ansible (devops server)

```
$ ansible-playbook -i ./inventory.ini gitlab.yml
```

### Run gitlab-executor (docker-based)

```
# curl -LJO "https://gitlab-runner-downloads.s3.amazonaws.com/latest/deb/gitlab-runner_amd64.deb"
# dpkg -i gitlab-runner_amd64.deb
# cp gitlab-runner/config.toml /etc/gitlab-runner/config.toml 
# systemctl restart gitlab-runner
   
```   
### Install k8s on k8s servers and setup
  
```
## k8s Staging example
$ curl -sfL https://get.k3s.io | sh -
$ sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/k8s-staging
$ sudo chmod 755 ~/.kube/k8s-staging
$ sed -i "s/127.0.0.1/192.168.1.100/"  ~/.kube/k8s-staging
$ export KUBECONFIG=~/.kube/k8s-staging
$ kubectl cluster-info

### Create k8s namespace
$ kubectl create namespace staging

### Setup GitLab registry credentials for k8s

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
### Create Gitlab project and push endurosat-cicd folder and create Gitlab CI/CD variables for this repo 






