# offlinehacker/openvpn-k8s

- [Introduction](#introduction)
- [Contributing](#contributing)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration Parameters](#configuration-parameters)

# Introduction

Dockerfile to run [openvpn](https://openvpn.net/) inside [kubernetes](http://kubernetes.io/).

# Contributing

If you find this image useful you can help by doing one of the following:

- *Send a Pull Request*: you can add new features to the docker image, which will be integrated into the official image.
- *Report a Bug*: if you notice a bug, please issue a bug report at [Issues](https://github.com/offlinehacker/openvpn-k8s/issues), so we can fix it as soon as possible.

# Installation

Automated builds of the image are available on [Dockerhub](https://hub.docker.com/r/offlinehacker/openvpn-k8s) and is the recommended method of installation.

```bash
docker pull offlinehacker/openvpn-k8s:latest
```

Alternatively you can build the image locally.

```bash
git clone https://github.com/offlinehacker/openvpn-k8s.git
cd openvpn
docker build -t offlinehacker/openvpn-k8s .
```

# Quick Start

This image was created to simply have openvpn access to kubernetes cluster.

First you will need to create [secret volume](http://kubernetes.io/v1.1/docs/user-guide/secrets.html) with dh params and server certificate in pkcs12 format.

- Create kubernetes secret volume:

`openvpn-secrets.yaml` file:

```
apiVersion: v1
kind: Secret
metadata:
  name: openvpn
data:
  dh.pem: <base64_encoded_dh_pem_file>
  certs.p12: <base64_encoded_certs_file>
```

    kubectl create -f openvpn-secrets.yaml

- Create kubernetes replication controller:

`openvpn-controller.yaml` file:

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: openvpn
  labels:
    name: openvpn
spec:
  replicas: 2
  selector:
    name: openvpn
  template:
    metadata:
      labels:
        name: openvpn
    spec:
      containers:
        - name: openvpn
          image: offlinehacker/openvpn-k8s
          securityContext:
            capabilities:
              add:
                - NET_ADMIN
          env:
            - name: OVPN_NETWORK
              value: 10.240.0.0
            - name: OVPN_SUBNET
              value: 255.255.0.0
            - name: OVPN_PROTO
              value: tcp
            - name: OVPN_K8S_SERVICE_NETWORK
              value: 10.241.240.0
            - name: OVPN_K8S_SERVICE_SUBNET
              value: 255.255.240.0
            - name: OVPN_K8S_DNS
              value: 10.241.240.10
          ports:
            - name: openvpn
              containerPort: 1194
          volumeMounts:
            - mountPath: /etc/openvpn/pki
              name: openvpn
      volumes:
        - name: openvpn
          secret:
            secretName: openvpn
```

- Create kubernetes service:

`openvpn-service.yaml` file:

```
kind: Service
apiVersion: v1
metadata:
  name: openvpn
spec:
  ports:
    - name: openvpn
      port: 1194
      targetPort: 1194
  selector:
    name: openvpn
  type: LoadBalancer
```

    kubectl create -f openvpn-service.yaml

# Configuration Parameters

Below is the complete list of available options that can be used to customize your packetbeat container instance.

- **OVPN_NETWORK**: Network allocated for openvpn clients (default: 10.240.0.0).
- **OVPN_SUBNET**: Network subnet allocated for openvpn client (default: 255.255.0.0).
- **OVPN_PROTO**: Protocol used by openvpn tcp or udp (default: udp).
- **OVPN_NATDEVICE**: Device connected to kuberentes service network (default: eth0).
- **OVPN_K8S_ROUTES**: Comma separated routes for NAT with possibility to specify ports (example: 10.3.0.0/24:53/udp)
- **OVPN_MULTIPLE_CERTS**: Allow multiple connections with a specific certificate
- **OVPN_K8S_SERVICE_NETWORK**: Kubernetes service network (required).
- **OVPN_K8S_SERVICE_SUBNET**: Kubernetes service network subnet (required).
- **OVPN_K8S_DOMAIN**: Kuberentes cluster domain (default: cluster.local).
- **OVPN_K8S_DNS**: Kuberentes cluster dns server (required).
- **OVPN_K8S_DH**: Openvpn dh.pem file path (default: /etc/openvpn/pki/dh.pem).
- **OVPN_K8S_CERTS**: Openvpn certs.p12 file path (default: /etc/openvpn/pki/certs.p12).
