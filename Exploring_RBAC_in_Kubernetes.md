# Exploring RBAC in Kubernetes

![RBAC in Kubernetes](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/rbac2.png "RBAC in Kubernetes")

## References:
* https://coreos.com/blog/hands-on-with-rbac-in-kubernetes-1.8
* http://blog.kubernetes.io/2017/04/rbac-support-in-kubernetes.html
* https://kubernetes.io/docs/admin/authorization/rbac/
* https://blog.openshift.com/understanding-service-accounts-sccs/

## Observations

Tested with:

- Minishift v1.11.0+4459917
- Kubernetes 3.7
- istio 0.2.7
- VirtualBox 5.1.30
- macOS High Sierra, version 10.13.2 (17C88)

## RBAC

```bash
$ oc create namespace dev
```

```bash
$ oc get sa -n dev
NAME       SECRETS   AGE
builder    2         12s
default    2         12s
deployer   2         12s
```

```bash
$ oc get clusterroles -n dev
admin
basic-user
cluster-admin
cluster-debugger
cluster-reader
cluster-status
edit
istio-ca-istio-system
istio-initializer-istio-system
istio-mixer-istio-system
istio-pilot-istio-system
istio-sidecar-istio-system
...
sudoer
...
system:node
...
system:router
...
view
```

```bash
$ oc describe clusterrole/view -n dev
Name:			view
Created:		7 hours ago
Labels:			<none>
Annotations:		openshift.io/description=A user who can view but not edit any resources within the project. They can not view secrets or membership.
			openshift.io/reconcile-protect=false
Verbs			Non-Resource URLs	Resource Names	API Groups			Resources
[get list watch]	[]			[]		[]				[configmaps endpoints persistentvolumeclaims pods replicationcontrollers serviceaccounts services]
[get list watch]	[]			[]		[]				[bindings events limitranges namespaces namespaces/status pods/log pods/status replicationcontrollers/status resourcequotas resourcequotas/status]
[get list watch]	[]			[]		[autoscaling]			[horizontalpodautoscalers]
[get list watch]	[]			[]		[batch]				[cronjobs jobs scheduledjobs]
[get list watch]	[]			[]		[extensions]			[deployments deployments/scale horizontalpodautoscalers replicasets replicasets/scale]
[get list watch]	[]			[]		[extensions]			[daemonsets]
[get list watch]	[]			[]		[apps]				[deployments deployments/scale statefulsets]
[get list watch]	[]			[]		[ build.openshift.io]		[buildconfigs buildconfigs/webhooks builds]
[get list watch]	[]			[]		[ build.openshift.io]		[builds/log]
[view]			[]			[]		[build.openshift.io]		[jenkins]
[get list watch]	[]			[]		[ apps.openshift.io]		[deploymentconfigs deploymentconfigs/scale]
[get list watch]	[]			[]		[ apps.openshift.io]		[deploymentconfigs/log deploymentconfigs/status]
[get list watch]	[]			[]		[ image.openshift.io]		[imagestreamimages imagestreammappings imagestreams imagestreamtags]
[get list watch]	[]			[]		[ image.openshift.io]		[imagestreams/status]
[get]			[]			[]		[ project.openshift.io]		[projects]
[get list watch]	[]			[]		[ quota.openshift.io]		[appliedclusterresourcequotas]
[get list watch]	[]			[]		[ route.openshift.io]		[routes]
[get list watch]	[]			[]		[ route.openshift.io]		[routes/status]
[get list watch]	[]			[]		[ template.openshift.io]	[processedtemplates templateconfigs templateinstances templates]
[get list watch]	[]			[]		[ build.openshift.io]		[buildlogs]
[get list watch]	[]			[]		[]				[resourcequotausages]
```

### 1. User Accounts


```bash
$ oc create rolebinding roger-rb-view --clusterrole view --user roger-ua -n dev
rolebinding "roger-rb-view" created
```

```bash
$ oc auth can-i get pods -n dev --as roger-ua
yes
```

```bash
$ oc auth can-i create deployments -n dev --as roger-ua
no - User "roger-ua" cannot create deployments.extensions in project "dev"
```

### 2. Service Accounts


sa-read-deployments
sa-write-deployments

$ kubectl create serviceaccount --namespace dev sa-read-deployments
$ kubectl create serviceaccount --namespace dev sa-write-deployments
