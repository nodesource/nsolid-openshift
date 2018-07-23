{N|Solid, Docker, and OpenShift logo}

## Overview

This repository is for deploying [N|Solid](https://nodesource.com/products/nsolid) with [OpenShift](http://openshift.com/). It assumes that OpenShift is already setup for your environment.

{N|Solid, Docker, and OpenShift diagram}

### Table of Contents
- [Installing OpenShift Origin](#a1)
- [Quickstart](#a2)
    - [Access N|Solid Dashboard](#a3)
    - [Uninstall N|Solid](#a4)
- [Deploy Sample App with N|Solid](#a5)
- [Production Install](#a6)
    - [N|Solid namespace](#a7)
    - [nginx SSL certificates](#a8)
    - [Basic Auth file](#a9)
    - [Secret object for certs](#a10)
    - [Configmap object for settings](#a11)
    - [Define Services](#a12)
- [Debugging / Troubleshooting](#a15)
    - [Configuring Apps for N|Solid with OpenShift](#a16)
        - [Buiding an N|Solid app](#a17)
            - [Docker](#a18)
            - [OpenShift](#a19)
        - [Accessing your App](#a20)
    - [Accessing N|Solid object in OpenShift](#a21)
- [License & Copyright](#a28)

<a name="a1"/>

## Installing OpenShift

* [local with OpenShift Origin](./docs/install/local.md) - for local development / testing.
* [OpenShift Online](http://openshift.com/) - RedHat OpenShift Online

<a name="a2"/>

## Quickstart

Make sure your `oc` is pointing to your active cluster and you are logged in with admin rights.

```bash
./install
```

This command will install the N|Solid Console, the N|Solid Storage service, and a secure HTTPS proxy to the `nsolid` project.

It can take a little while for OpenShift to download the N|Solid Docker images.  You can verify
that they are active by running:

```bash
oc get pods
```

When all three pods (console, storage, and nginx-secure-proxy) have a status of 'Running', you may
continue to access the N|Solid Console.

<a name="a3"/>

### Access N|Solid Console

#### Get the console's URL

```bash
oc describe routes nsolid-console
```

You'll get output similar to:

```bash 
Name:		 nsolid-console
Namespace:	 nsolid
Created:	 7 minutes ago
Labels:		 app=nginx-secure-proxy
Annotations:	 openshift.io/host.generated=true
Requested Host:	 nsolid-console-nsolid.10.1.10.16.xip.io
		   exposed on router router 7 minutes ago
Path:		 <none>
TLS Termination: passthrough
Insecure Policy: <none>
Endpoint Port:	 10443

Service:	 nginx-secure-proxy
Weight:		 100 (100%)
Endpoints:	 x.x.x.x:10080, x.x.x.x:10443
```

The URL is the `Requested Host` value. In this example it is `https://nsolid-console-nsolid.10.1.10.16.xip.io`. It
will be different on your local machine.

**NOTE:** You will need to ignore the security warning on the self signed certificate to proceed.

#### Secure credentials

* Default username: `nsolid`
* Default password: `demo`


![Welcome Screen](./docs/images/welcome.png)

<a name="a4"/>

### Uninstall N|Solid from OpenShift cluster

```bash
oc delete ns nsolid --cascade
```

<a name="a5"/>

## Deploy Sample App with N|Solid

### Quick Start

```bash
cd sample-app
./install
```

This will build the sample app Docker container locally, and then create the `sample-app` project and services.

**NOTE:** container image in `sample-app.deployment.yml` assumes `sample-app:v1` docker image. This will work if you're using OpenShift locally.

If you are working in a cloud environment, you will need to push the sample-app to a public Docker registry
like [Dockerhub](https://dockerhub.com) or [Quay.io](https://quay.io), and update the sample-app Deployment file.

#### Get the sample app's URL

```bash
oc describe routes sample-app
```

You'll get output similar to:

```bash 
Name:			sample-app
Namespace:		sample-app
Created:		3 minutes ago
Labels:			<none>
Annotations:		openshift.io/host.generated=true
Requested Host:		sample-app-sample-app.10.1.10.16.xip.io
			  exposed on router router 3 minutes ago
Path:			<none>
TLS Termination:	<none>
Insecure Policy:	<none>
Endpoint Port:		<all endpoint ports>

Service:	sample-app
Weight:		100 (100%)
Endpoints:	x.x.x.x:4444
```

The URL is the `Requested Host` value. In this example it is `http://sample-app-sample-app.10.1.10.16.xip.io`. It
will be different on your local machine.

<a name="a6"/>

## Production Install

**NOTE:** Assumes `oc` is configured and pointed at your OpenShift cluster properly.

<a name="a7"/>

#### Create the `nsolid` project to help isolate and manage the N|Solid components.

```
oc new-project nsolid --description="N|Solid Console and Storage" --display-name="N|Solid"
```

<a name="a8"/>

#### Create nginx SSL certificates

```
openssl req -x509 -nodes -newkey rsa:2048 -keyout conf/certs/nsolid-nginx.key -out conf/certs/nsolid-nginx.crt
```

<a name="a9"/>

#### Create Basic Auth file

```
rm ./conf/nginx/htpasswd
htpasswd -cb ./conf/nginx/htpasswd {username} {password}
```

<a name="a10"/>

#### Create a `secret`  for certs to mount in nginx

```
oc create secret generic nginx-tls --from-file=conf/certs --namespace=nsolid
```

<a name="a11"/>

#### Create `configmap` for nginx settings
```
oc create configmap nginx-config --from-file=conf/nginx --namespace=nsolid
```

<a name="a12"/>

#### Define the services

```bash
oc create -f conf/nsolid.services.yml
```

#### Create persistent disks

N|Solid components require persistent storage.  OpenShift does not (yet!)
automatically handle provisioning of disks consistently across all cloud providers.
As such, you will need to manually create the persistent volumes.

<a name="a15"/>

## Debugging / Troubleshooting

<a name="a16"/>

### Configuring Apps for N|Solid with OpenShift

<a name="a17"/>

#### Buiding an N|Solid app

<a name="a18"/>

##### Docker

Make sure your docker image is build on top of an N|Solid image based on the correct version of Node.js for your project:

```dockerfile
FROM nodesource/nsolid:carbon-latest
```


<a name="a19"/>

##### OpenShift

When defining your application make sure the following `ENV` are set.

```yaml
  env:
    - name: NSOLID_APPNAME
      value: sample-app
    - name: NSOLID_COMMAND
      value: "storage.nsolid:9001"
    - name: NSOLID_DATA
      value: "storage.nsolid:9002"
    - name: NSOLID_BULK
      value: "storage.nsolid:9003"
```

Optional flags:

```yaml
  env:
    - name: NSOLID_TAGS
      value: "nsolid-carbon,staging"
```

A comma seperate list of tags that can be used to filter processes in the N|Solid console.

<a name="a20"/>

#### Accessing your App

To access your application outside of the OpenShift cluster, you must create a route to your application.
You can refer to the `sample-app.route.yml` file as an example for creating it via files, or use the OpenShift
console to create the route manaully. 

<a name="a21"/>

### Accessing N|Solid object in OpenShift

After creating the sample app project, your context is switched to that project. You must switch
the active project back to nsolid to manage those resources.

```bash
oc project nsolid
```

<a name="a28" />

## License & Copyright

**nsolid-openshift* is Copyright (c) 2016 NodeSource and licensed under the MIT license. All rights not explicitly granted in the MIT license are reserved. See the included [LICENSE.md](LICENSE.md) file for more details.
