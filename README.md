# Exploring API Mesh with OpenShift, Weave Scope and Istio

We are going to try solving below questions by exploring a local Kubernetes, APIs, injecting sidecars and playing with Istio samples:

1. Is API Mesh a kind of Distributed Application?
2. Why Istio is required in a Containerized Platform?
3. Why Deployment Container Patterns (sidecar, ambassador, adapter) are important?
4. How we have to implement:  
   * Monitoring
   * Observability
   * Security  

## Observations

Tested with:

- Ansible 2.3+
- minishift v1.11.0+4459917
- kubernetes 3.7
- istio 0.2.7
- VirtualBox 5.1.30
- macOS High Sierra, version 10.13.2 (17C88)

## Prerequisites

- Minishift Ansible Role: https://galaxy.ansible.com/chilcano/minishift
- Weave Scope Ansible Role: https://galaxy.ansible.com/chilcano/weave-scope
- Istio Ansible Role: https://galaxy.ansible.com/chilcano/istio
- `sudo` access in your host is required for installing packages.

## 1. Getting step-by-step a full API Mesh to test locally with OpenShift, Weave Scope and Istio.

Install the roles:
```
$ sudo ansible-galaxy install chilcano.minishift
$ sudo ansible-galaxy install chilcano.weave-scope
$ sudo ansible-galaxy install chilcano.istio
```

Download the playbook and create the `inventory`:
```
$ git clone https://github.com/chilcano/ansible-minishift-istio-security
$ cd ansible-minishift-istio-security
$ echo $(hostname) > ./inventory
```

Update the playbooks accordingly, for example, update:
```yaml
  ...
  iso-url: https://github.com/minishift/minishift-b2d-iso/releases/download/v1.2.0/minishift-b2d.iso
  profile: openshift0
  openshift-version: "v3.7.0"
  ...
  release_tag_name: "" # latest
  ...
```

### Step 1: Creating a minimalist OpenShift Cluster in a VM.

```
$ ansible-playbook -i inventory 00a-minishift.yml -e vm=openshift1 --ask-become-pass
```

#### Explanation

1. The Minishift Ansible Role will download and install all components required to get an OpenShift Cluster running in a VM.
2. The `-e vm=openshift1` means that `openshift1` is the name of the Minishift instance.
3. The Ansible Playbook executed (`00a-minishift.yml`) and the Ansible Role used (`chilcano.minishift`) will login in the recently created OpenShift instance through the `oc` client. Once done, you can login to OpenShift Web Console by using `http://openshift1:8443`. It's very important to accept and trust all TLS/SSL Certificates.

### Step 2: Deploying Weave Scope in OpenShift.

```
$ ansible-playbook -i inventory 00b-weavescope.yml -e vm=openshift1 --ask-become-pass
```

#### Explanation

1. An OpenShift running instance is required, also an Kubernetes account and permissions. By default this Ansible Role will use `system:admin` account.
2. The Weave Scope Ansible Role will download and apply the Kubernetes Deployment YAML file to deploy Weave Scope as an app in the OpenShift.
3. The installation of Weave Scope is not mandatory. It's a good tool to monitor, manage and visualize the OpenShift Cluster, Pods and Containers.

### Step 3: Deploying Istio and BookInfo App.

```
$ ansible-playbook -i inventory 00c-istio.yml -e vm=openshift1 --ask-become-pass
```


#### Explanation

1. The Istio Ansible Role will download and deploy Istio, Istio addons (prometheus, graphana, zipkin and servicegraph) on an OpenShift running locally.
2. Also the Istio Ansible Role will deploy the BookInfo App in OpenShift and will execute `istioctl` to inject sidecars (Envoy Proxy - https://www.envoyproxy.io) in every Pod.
3. Once deployed the BookInfo we have to wait for few seconds in order to use the `BookInfo` App. To check that, follow next commands:

```sh
$ minishift status
Minishift:  Running
Profile:    openshift1
OpenShift:  Running (openshift v3.7.0+7ed6862)
DiskUsage:  29% of 17.9G

$ eval $(minishift oc-env)

$ oc project bookinfo
Now using project "bookinfo" on server "https://192.168.99.101:8443".

$ oc status
In project bookinfo on server https://192.168.99.101:8443

svc/details - 172.30.229.55:9080
  pod/details-v1-1464079269-2g4zf runs istio/examples-bookinfo-details-v1:0.2.3, docker.io/istio/proxy_debug:0.2.7

svc/productpage - 172.30.99.163:9080
  pod/productpage-v1-3915871613-mc87n runs istio/examples-bookinfo-productpage-v1:0.2.3, docker.io/istio/proxy_debug:0.2.7

svc/ratings - 172.30.96.18:9080
  pod/ratings-v1-327106889-p8hz7 runs istio/examples-bookinfo-ratings-v1:0.2.3, docker.io/istio/proxy_debug:0.2.7

svc/reviews - 172.30.179.156:9080
  pod/reviews-v3-1994447391-r9mfn runs istio/examples-bookinfo-reviews-v3:0.2.3, docker.io/istio/proxy_debug:0.2.7
  pod/reviews-v1-3806695627-6swvd runs istio/examples-bookinfo-reviews-v1:0.2.3, docker.io/istio/proxy_debug:0.2.7
  pod/reviews-v2-3096629009-jq6tp runs istio/examples-bookinfo-reviews-v2:0.2.3, docker.io/istio/proxy_debug:0.2.7

View details with 'oc describe <resource>/<name>' or list everything with 'oc get all'.

$ oc get pods
NAME                              READY     STATUS            RESTARTS   AGE
details-v1-1464079269-n75st       0/2       PodInitializing   0          7m
productpage-v1-3915871613-hl68p   0/2       PodInitializing   0          7m
ratings-v1-327106889-4c6cs        0/2       PodInitializing   0          7m
reviews-v1-3806695627-44qkz       0/2       PodInitializing   0          7m
reviews-v2-3096629009-d7r76       0/2       PodInitializing   0          7m
reviews-v3-1994447391-dd7vs       0/2       PodInitializing   0          7m
```

4. If all Pods have `Running` as status, then you can use the BookInfo App with Istio (API Mesh). Just open you browser with this URL `http://istio-ingress-istio-system.192.168.99.101.nip.io/productpage`. Where the IP address for `openshift1` is `192.168.99.101`, change it if required.

## 2. Other Sample Ansible Playbooks to explore.

If you have cloned this repository `https://github.com/chilcano/ansible-minishift-istio-security` then you will see other playbooks:

### 2.1. Install Openshift, deploy Weave Scope, Istio and BookInfo App.
```
$ ansible-playbook -i inventory 01-minishift-weavescope-istio.yml -e vm=openshift2 --ask-become-pass
```

### 2.2. Remove Weave Scope, Istio and BookInfo App.
```
$ ansible-playbook -i inventory 02a-remove-weavescope-istio.yml -e vm=openshift2 --ask-become-pass
```

### 2.3. Remove the Minishift instance (VM).
```
$ ansible-playbook -i inventory 02b-remove-minishift.yml -e vm=openshift2 --ask-become-pass
```

## Screenshots

1. OpenShift, BookInfo App, Istio and Weave Scope.

![OpenShift, BookInfo App, Istio and Weave Scope](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-1-openshift.png "OpenShift, BookInfo App, Istio and Weave Scope")

2. Exploring OpenShift with Weave Scope.

![Exploring OpenShift with Weave Scope](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-2-weave-scope.png "Exploring OpenShift with Weave Scope")

3. BookInfo App deployed on OpenShift.

![BookInfo App deployed on OpenShift](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-3-istio-bookinfo-app.png "BookInfo App deployed on OpenShift")

4. Tracing with Zipkin.

![Tracing with Zipkin](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-4-istio-zipkin.png "Tracing with Zipkin")

5. Exploring metrics with Grafana.

![Exploring metrics with Grafana](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-5-istio-grafana.png "Exploring metrics with Grafana")

6. Viewing the flows with ServiceGraph.

![Viewing the flows with ServiceGraph](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-6-istio-servicegraph.png "Viewing the flows with ServiceGraph")

7. Selecting `istio-system` namespace.

![Selecting istio-system namespace](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-7-weave-scope-istio-system.png "Selecting istio-system namespace")

8. Selecting `bookinfo` namespace.

![Selecting bookinfo namespace](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-8-weave-scope-bookinfo.png "Selecting bookinfo namespace")

9. Exploring in depth the API Mesh.

![Exploring in depth the API Mesh](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-9-weave-scope-bookinfo-mesh.png "Exploring in depth the API Mesh")
