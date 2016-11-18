## Setup

### Prerequisites

#### Docker

You must have Docker installed for your platform:

* [Docker](https://docs.docker.com/engine/installation/linux/)
* [Docker for Mac](https://docs.docker.com/engine/installation/mac/)
* [Docker for Windows](https://docs.docker.com/engine/installation/windows/)

#### The oc command line tool

Install the proper version of `oc` for your environment.  oc is used to interact with a running
OpenShift cluster.

[OpenShift Origin Releases](https://github.com/openshift/origin/releases/tag/v1.3.1)

#### Starting OpenShift Origin

Once you have the `oc` client locally, you can use it to start a local OpenShift cluster on your development machine.

```bash
oc cluster up --version=v1.3.1
```

### OpenShift Dashboard

Once the OpenShift cluster is up, you will see the URL for the Dashboard along with the credentials to use.

```bash
   OpenShift server started.
   The server is accessible via web console at:
       https://x.x.x.x:8443

   You are logged in as:
       User:     developer
       Password: developer

   To login as administrator:
       oc login -u system:admin
```

It is recommended to run `oc login -u system:admin` after the cluster is setup so that you have the proper permissions
to create a new project.

### Permissions after creating a new project

When new projects are created, the `developer` account needs to be granted rights to see it in the Dashboard.

```bash
oc policy add-role-to-user admin developer -n {projectname}
```

### Stopping OpenShift

```bash
oc cluster down
```
