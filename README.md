# Exploring Service Mesh with OpenShift, Weave Scope and Istio

Basically we are going to learn and understand what is Service Mesh by exploring a local Kubernetes, APIs, injecting sidecars and playing with Istio samples; in other words, we are going to try solving below questions:

1. Is Service Mesh a kind of Distributed Application?
2. Why Istio is required in a Containerized Platform?
3. Why Deployment Container Patterns (sidecar, ambassador, adapter) are important?
4. How we have to implement:  
   * Monitoring
   * Observability
   * Security  

![Exploring Service Mesh - OpenShift with Weave Scope and Istio](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-2-weave-scope.png "Exploring Service Mesh - OpenShift with Weave Scope and Istio")

## Observations

Tested with:

- Ansible 2.3+
- Minishift v1.11.0+4459917
- Kubernetes 3.7
- istio 0.2.7
- VirtualBox 5.1.30
- macOS High Sierra, version 10.13.2 (17C88)

## Prerequisites

- Chilcano's Ansible Roles:
   * Minishift (https://galaxy.ansible.com/chilcano/minishift)
   * Weave Scope (https://galaxy.ansible.com/chilcano/weave-scope)
   * Istio (https://galaxy.ansible.com/chilcano/istio)
- `sudo` access in your Host for installing packages.

## 1. Getting step-by-step a full Service Mesh to test locally with OpenShift, Weave Scope and Istio.

Download from Github the sample playbooks and create the `inventory`:
```
$ git clone https://github.com/chilcano/ansible-minishift-istio-security
$ cd ansible-minishift-istio-security
$ echo $(hostname) > ./inventory
```

Install the Chilcano's Ansible Roles:
```
$ sudo ansible-galaxy install -r requirements.yml
```

Where `requirements.yml` is:
```yaml
- src: chilcano.minishift
- src: chilcano.istio
- src: chilcano.weave-scope
```

### Step 1: Creating a minimalist OpenShift Cluster in a VM.

```
$ ansible-playbook -i inventory install-minishift.yml -e vm=openshift1 --ask-become-pass
```

#### Explanation

1. The Minishift Ansible Role will download and install all components required to get an OpenShift Cluster running in a VM.
2. The `-e vm=openshift1` means that `openshift1` is the name of the Minishift instance.
3. The Ansible Playbook executed (`install-minishift.yml`) and the Ansible Role used (`chilcano.minishift`) will login in the recently created OpenShift instance through the `oc` client. Once done, you can login to OpenShift Web Console by using `http://openshift1:8443`. It's very important to accept and trust all TLS/SSL Certificates.

### Step 2: Deploying Weave Scope in OpenShift.

```
$ ansible-playbook -i inventory install-weavescope.yml -e vm=openshift1 --ask-become-pass
```

#### Explanation

1. An OpenShift running instance is required, also an Kubernetes account and permissions. By default this Ansible Role will use `system:admin` account.
2. The Weave Scope Ansible Role will download and apply the Kubernetes Deployment YAML file to deploy Weave Scope as an app in the OpenShift.
3. The installation of Weave Scope is not mandatory. It's a good tool to monitor, manage and visualize the OpenShift Cluster, Pods and Containers.


In order to get access to Weave Scope from browser, we should forward the Weave Scope's port to the Host's port.
Considering the Weave Scope App listens, by default, on the port `4040`, then to forward to host's port on `4040` to use next command:
```
$ oc port-forward -n weave-scope "$(oc get -n weave-scope pod --selector=weave-scope-component=app -o jsonpath='{.items..metadata.name}')" 4040:4040

Forwarding from 127.0.0.1:4040 -> 4040
Forwarding from [::1]:4040 -> 4040
Handling connection for 4040
Handling connection for 4040
Handling connection for 4040
Handling connection for 4040
...
```

To forward to host's port on `4041`, send logs to log file and run it on background to use the next command:
```
$ oc port-forward -n weave-scope "$(oc get -n weave-scope pod --selector=weave-scope-component=app -o jsonpath='{.items..metadata.name}')" 4041:4040 > openshift-scope.log &
[1] 3087

$ tail -f openshift-scope.log
Forwarding from 127.0.0.1:4041 -> 4040
Forwarding from [::1]:4041 -> 4040
Handling connection for 4041
Handling connection for 4041
Handling connection for 4041
Handling connection for 4041
Handling connection for 4041
...
```

To kill the `oc port-forward ...` command, to do this:
```
$ killall oc
```
Or
```
$ kill 3087
```

Once done, open your browser with this URL and you could visualize all Pods, Containers, Controllers, etc. of your OpenShift Cluster.

### Step 3: Deploying Istio and BookInfo App.

```
$ ansible-playbook -i inventory install-istio.yml -e vm=openshift1 --ask-become-pass
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

4. If all Pods have `Running` as status, then you can use the BookInfo App with Istio (Service Mesh). Just open you browser with this URL `http://istio-ingress-istio-system.192.168.99.101.nip.io/productpage`. Where the IP address for `openshift1` is `192.168.99.101`, change it if required.


### Step 4: Re-starting the OpenShift VM.

Sometimes the OpenShift's VM can not be running because you have rebooted your computer or have stoppet the VM from VirtualBox, oVirt, VMWare, etc.
In that case, in order to re-install everything you just need trigger the `$ minishift start` command, and to do easier you can use the previous playbook used to create the OpenShift's VM with the `action_to_trigger` param set to `install` or external param `do=install` from command line. Although, the default value for `action_to_trigger` is `install`, that is useful if you want to trigger other actions like `install`, `fresh_install` or `clean`.

```
$ ansible-playbook -i inventory install-minishift.yml -e vm=openshift1 -e do=install --ask-become-pass
```

## 2. Other Sample Ansible Playbooks to explore.

If you have cloned this repository `https://github.com/chilcano/ansible-minishift-istio-security` then you will see other playbooks:

### 2.1. Install Openshift, deploy Weave Scope, Istio and BookInfo App.
```
$ ansible-playbook -i inventory install-minishift-weavescope-istio.yml -e vm=openshift2 --ask-become-pass
```

### 2.2. Remove Weave Scope, Istio and BookInfo App.
```
$ ansible-playbook -i inventory remove-weavescope-istio.yml -e vm=openshift2 --ask-become-pass
```

### 2.3. Remove the Minishift instance (VM).
```
$ ansible-playbook -i inventory remove-minishift.yml -e vm=openshift2 --ask-become-pass
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

9. Exploring in depth the Service Mesh.

![Exploring in depth the Service Mesh](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-9-weave-scope-bookinfo-mesh.png "Exploring in depth the Service Mesh")

## References about Service Mesh and Istio

1. Catch up on the Istio and service mesh excitement from Kubecon 2017! (by Lin Sun)
   * https://developer.ibm.com/dwblog/2017/istio-service-mesh-kubecon-news-announcements
2. Istio Workshop (includes Istio installation for Google Cloud and AWS)
   * https://github.com/retroryan/istio-workshop (by Ryan Knight)
   * https://github.com/ZackButcher/istio-workshop (by Zack Butcher)
   * https://github.com/ipedrazas/istioworkshop-cc (by Ivan Pedrazas)
3. Deep Dive Envoy and Istio Workshop (by Christian Posta)
   * http://blog.christianposta.com/microservices/deep-dive-envoy-and-istio-workshop/
4. Istio Traffic Management – Diving Deeper (by Ricardo Lourenco)
   * https://blog.openshift.com/istio-traffic-management-diving-deeper
