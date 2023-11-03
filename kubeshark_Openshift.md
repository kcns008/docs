## Openshift Cluster Spin up and Kubeshark Deployment 


#### install rosa from https://console.redhat.com/openshift/downloads Based on you PC download tar and move to PATH / bin folder

```shell
tar xvf rosa-linux.tar.gz
```
```shell
sudo mv rosa /usr/local/bin/rosa
```

#### Check for latest rosa version

```shell
rosa version
```
#### rosa autocompletion if using zsh, helpful but not requires
```shell
rosa completion zsh > "${fpath[1]}/_rosa"
```

#### Get rosa token from https://console.redhat.com/openshift/token/rosa
```shell
rosa login --token="eyJh..."
```


#### Configure AWS CLI if not already, if need help see https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html
```shell
aws configure
```

#### Create Require Roles in AWS for Openshift Cluster
```shell
rosa create account-roles --mode auto
```

#### To start spinning up Openshift cluster
```shell
rosa create cluster --cluster-name kubeshark --sts --mode auto
```

#### it will ask for Role to select `ManagedOpenShift-Installer-Role` , Wait for around 40 - 45 Minutes, you can track progress through `rosa describe cluster -c kubeshark`

#### Once Openshift Cluster Up and running create `cluster-admin` user
```shell
rosa create admin --cluster=kubeshark
```

#### you will get `oc login` something like this, assuming you have `oc cli` install, if not get it from https://console.redhat.com/openshift/downloads
```shell
oc login https://api.kubeshark.ABC1.p1.openshiftapps.com:6443 --username cluster-admin --password <super_long_pwd>
```

#### Just to verify all nodes `READY` run
```shell
oc get nodes
```

#### Now Prepare `default` namespace in Openshift cluster for Kubeshare by allowing kubeshark POD spinup will `privileged` and `anyuid` SCC by assigning those SCC to SA `default` and `kubeshark-service-account`
```shell
oc adm policy add-scc-to-user privileged -z default -n default
```
```shell
oc adm policy add-scc-to-user anyuid -z default -n default
```
```shell
oc adm policy add-scc-to-user privileged -z kubeshark-service-account -n default
```
```shell
oc adm policy add-scc-to-user anyuid -z kubeshark-service-account -n default
```

#### install `kubeshark` through `brew` if not already
```shell
brew tap kubeshark/kubeshark
```
```shell
brew install kubeshark
```

#### Deploy `kubeshark` , it will open Local Dashboard with traffic

```shell
kubeshark tap
```

#### if you want to return / start gain `kubeshark` local dashboard
```shell
kubeshark proxy
```
