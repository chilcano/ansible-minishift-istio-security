# Exploring API Mesh with Openshift, Weave Scope and Istio

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

## Getting an API Mesh to test locally

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

### 1. Openshift Cluster and Weave-Scope

```
$ ansible-playbook -i inventory --ask-become-pass 01-minishift-weavescope.yml
```

#### Explanation

1. The Minishift Ansible Role will download and install all components required to get an OpenShift Cluster running in a VM.
2. The Weave Scope Ansible Role will download and apply the Kubernetes Deployment YAML file to deploy Weave Scope as an app in the OpenShift.


### 2. Openshift Cluster, Weave-Scope, Istio and BookInfo App

```
$ ansible-playbook -i inventory --ask-become-pass 02-minishift-weavescope-istio.yml
```

#### Explanation

1. The Minishift Ansible Role will download and install all components required to get an OpenShift Cluster running in a VM.
2. The Istio Ansible Role will download and deploy Istio, Istio addons (prometheus, graphana, zipkin and servicegraph) on an OpenShift running locally.
3. Also the Istio Ansible Role will deploy the BookInfo App in OpenShift and will execute `istioctl` to inject sidecars (Envoy Proxy - https://www.envoyproxy.io).

#### Screenshots

1. Openshift, BookInfo App, Istio and Weave Scope.

![Openshift, BookInfo App, Istio and Weave Scope](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-1-openshift.png "Openshift, BookInfo App, Istio and Weave Scope")

2. Exploring Openshift with Weave Scope.

![Exploring Openshift with Weave Scope](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-2-weave-scope.png "Exploring Openshift with Weave Scope")

3. BookInfo App deployed on Openshift.

![BookInfo App deployed on Openshift](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-3-istio-bookinfo-app.png "BookInfo App deployed on Openshift")

4. Tracing with Zipkin.

![Tracing with Zipkin](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-4-istio-zipkin.png "Tracing with Zipkin")


5. Exploring metrics with Grafana.

![Exploring metrics with Grafana](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-5-istio-grafana.png "Exploring metrics with Grafana")

6. Viewing the flows with ServiceGraph.

![Viewing the flows with ServiceGraph](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-6-istio-servicegraph.png "Viewing the flows with ServiceGraph")

7. Selecting `istio-system` namespace.

![Selecting istio-system namespace](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-7-weave-scope-istio-system.png "Selecting istio-system namespace")

8. Selecting `istio-system` namespace.

![Selecting bookinfo namespace](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-8-weave-scope-bookinfo.png "Selecting bookinfo namespace")

9. Exploring in depth the API Mesh.

![Exploring in depth the API Mesh](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-9-weave-scope-bookinfo-mesh.png "Exploring in depth the API Mesh")

## License

MIT / BSD
