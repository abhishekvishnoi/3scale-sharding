
Copy
# Router Sharding in OpenShift at the Namespace Level

Router sharding in OpenShift at the namespace level involves configuring multiple routers (HAProxy instances) to handle traffic for specific namespaces. This helps isolate traffic, improve performance, and provide better resource management. Below are the steps to perform router sharding in OpenShift at the namespace level.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Steps to Perform Router Sharding](#steps-to-perform-router-sharding)
   - [Create a New Router for Sharding](#1-create-a-new-router-for-sharding)
   - [Configure the Router Selector](#2-configure-the-router-selector)
   - [Create Routes in the Namespace](#3-create-routes-in-the-namespace)
   - [Verify Router Sharding](#4-verify-router-sharding)
   - [Optional: Scale the Router](#5-optional-scale-the-router)
   - [Repeat for Additional Namespaces](#6-repeat-for-additional-namespaces)
3. [Summary](#summary)
4. [License](#license)

---

## Prerequisites

Before proceeding, ensure the following:

- You have **cluster-admin privileges**.
- The OpenShift CLI (`oc`) is installed and configured.
- You have a running OpenShift cluster.

---

## Steps to Perform Router Sharding

### 1. Create a New Router for Sharding

You need to create a new router specifically for the namespace(s) you want to shard.

#### Step 1.1: Create a Router Deployment

Use the `oc adm router` command to create a new router deployment. Specify a unique name for the router and the namespace it will serve.

```bash
oc adm router <router-name> --replicas=1 --service-account=router --selector="router=<router-name>" --namespace=<namespace>
```

#### Step 1.2:  Label the Namespace

Label the namespace to associate it with the router.

```bash
oc label namespace <namespace> router=<router-name>
```

### 2. Configure the Router Selector

The router uses a selector to determine which routes it should handle. Configure the router to only serve routes from the labeled namespace.

#### Step 2.1: Edit the Router Deployment

Edit the router deployment to add the ROUTE_LABELS environment variable.

```bash
oc set env dc/<router-name> ROUTE_LABELS="router=<router-name>"
```

This ensures the router only processes routes with the label router=<router-name>.


### 3. Create Routes in the Namespace

When creating routes in the namespace, ensure they are labeled to match the router selector.

#### Step 3.1: Create a Route

Create a route in the namespace and label it.

```bash
oc create route edge <route-name> --service=<service-name> --namespace=<namespace>
oc label route <route-name> router=<router-name> --namespace=<namespace>
```

### 4. Verify Router Sharding

Verify that the router is only serving routes from the specified namespace.

#### Step 4.1: Check Router Logs

Check the logs of the router to ensure it is only processing routes from the labeled namespace.


```bash
oc logs dc/<router-name> -n default
```

#### Step 4.2: Test the Route

Access the route from a browser or using curl to ensure it is working as expected.


## Summary

##### Create a new router for the namespace.

##### Label the namespace and routes to match the router selector.

##### Configure the router to only handle routes with the specified label.

##### Verify the setup and scale as needed.

This approach allows you to isolate traffic and improve performance by dedicating routers to specific namespaces.





















