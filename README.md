# API Mesh & Security



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
- Istio Ansible Role: https://galaxy.ansible.com/chilcano/istio
- `sudo` access in your host is required for installing packages.


## Getting an API Mesh to test locally

Install the roles:
```
$ ansible-galaxy install chilcano.minishift
$ ansible-galaxy install chilcano.istio
```

Create an `inventory` file
```
$ echo $(hostname) > ./inventory
```

Copy the `minishift-istio-sec-01.yml` playbook from this Git repo to the current working directory:
```
$ cat minishift-istio-sec-01.yml
```

```yaml

```

Run the playbook:
```
$ ansible-playbook -i inventory --ask-become-pass minishift-istio-sec-01.yml
```


### Explanation

1. Executing the Minishift Ansible Role will download and install all components required to get an OpenShift Cluster running in a VM.
2. Executing the Istio Ansible Role will download and deploy Istio, Istio addons (prometheus, graphana, zipkin and servicegraph) and Weave Scope (https://github.com/weaveworks/scope) on an OpenShift running locally.
3. Also the Istio Ansible Role will deploy the BookInfo App in OpenShift and will execute `istioctl` to inject sidecars (Envoy Proxy - https://www.envoyproxy.io).

## Screenshots

1. api-mesh-security-1-openshift.png

![Openshift, BookInfo App, Istio and Weave Scope](https://github.com/chilcano/ansible-minishift-istio-security/blob/master/imgs/api-mesh-security-1-openshift.png "Openshift, BookInfo App, Istio and Weave Scope")


## License

MIT / BSD

## Author Information

This role was created in 2017 by [Roger Carhuatocto](https://www.linkedin.com/in/rcarhuatocto), author of [HolisticSecurity.io Blog](https://holisticsecurity.io).
