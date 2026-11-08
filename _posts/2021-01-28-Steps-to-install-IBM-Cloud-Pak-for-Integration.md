---
title:  Steps to install IBM Cloud Pak for Integration on an OCP environment
date: 2021-01-28 17:17:50
categories:
  - Procedure
tags:
  - pd
  - cp4i
  - ocp
---

IBM Cloud Pak for Integration(aka CP4I) brings together proven, best-in-class capabilities to deliver a streamlined integration solution. CP4I offers many components, you can install some of them.

## 1. Prerequisites

### 1.1 Environment Preparation

- An existing OpenShift 4.6.x cluster, I used OCP v4.6.17 for this setup.
- The OpenShift cluster have defined appropriate storage classes that meet the requirements of the respective integration capability, can refer <a href="https://www.ibm.com/support/knowledgecenter/en/SSGT7J_20.4/system_reqs/storage.html" target="_blank">here</a> for more information.
- Added the IBM Operator Catalog and the IBM Common Services Catalog to your cluster, can refer <a href="https://www.ibm.com/support/knowledgecenter/en/SSGT7J_20.4/install/install_online_catalog_sources.html" target="_blank">here</a> for more information.

### 1.2 Installation Preparation

1. Ensure that your client have logged in to the web console server, otherwise can't issue `oc` commands on your client.

   ```bash
   # oc login
   ```

2. Ensure that can access OCP web console, running following command to check its URL.

   ```bash
   # oc -n openshift-console get route
   NAME        HOST/PORT                                        PATH   SERVICES    PORT    TERMINATION          WILDCARD
   console     console-openshift-console.apps.s07.mcocp.com            console     https   reencrypt/Redirect   None
   ```
   
   The console URL in this example is `https://console-openshift-console.apps.s07.mcocp.com/`.
   
3. To run software from the IBM Entitled Registry, you must supply your entitlement key as a Kubernetes pull secret.

   1) Obtain an Entitlement key from <a href="https://myibm.ibm.com/products-services/containerlibrary" target="_blank">IBM Container Library</a>.
   
   2) Adding an entitlement key to a namespace
   
      ```bash
      # oc create secret docker-registry ibm-entitlement-key \
      --docker-username=cp \
      --docker-password=<entitlement-key> \
      --docker-server=cp.icr.io \
      --namespace=openshift-operators
      ```

4. Add the required label to the default namespace

       # oc label namespace default 'network.openshift.io/policy-group=ingress'

## 2. Installing the IBM Cloud Pak for Integration operator

- Search "IBM Cloud Pak for Integration" operator on OperatorHub under all projects, install it. It will install other relevant operators together.

   - From the navigation pane of OCP web console, click `Operators` > `OperatorHub`.
   - In the All Items field, enter `IBM Cloud Pak for Integration`.
   - Click the `IBM Cloud Pak for Integration` tile, then click `Install` button.
   - Leave all options as default, click `Install` button.
    
- Deploying Common Service instance if not have

  - From the left navigation pane of OCP web console, click `Operators` > `Installed Operators`.
  - Click `IBM Cloud Platform Common Services` under project `ibm-common-services`.
  - From the tab `Common Service`, click the button `Create CommonService`.
  - Leave all other options as default, click `Create` button.

- Updating OpenShift authentication before installing the IAM Operator

  - After Common Service instance created, click the instance `common-service` to edit.
  - From the YAML tab, scroll down to `spec` section, add below content and then click `Save` button.
    
    ```yaml
    spec:
      services:
        - name: ibm-iam-operator
          spec:
            authentication:
              config:
                preferredLogin: ROKS
                roksEnabled: true
                roksURL: 'https://oauth-openshift.apps.s07.mcocp.com'
                roksUserPrefix: ''
    ```

## 3. Deploying Platform Navigator instance

- Deployment using the Red Hat OpenShift web console

  - From the left navigation pane of OCP web console, click `Operators` > `Installed Operators`.
  - Click `IBM Cloud Pak for Integration Platform Navigator` under project `openshift-operators`.
  - From the tab `Platform Navigator`, click the button `Create PlatformNavigator`.
  - Change the `License Accept` option to true.
  - Leave all other options as default, click `Create` button.

  > This phase might hit the issue#1 and issue#2, check the issues section for more information.

- Verifying the deployment progress, it will install Common UI too.

  From the instance, its status should be changed to "Condition: Ready".

      # oc get pod -n openshift-operators
      ...
      integration-navigator-ibm-inte-3c22-deployment-885cfb758-fxr8z   2/2     Running   0          9m43s
      integration-navigator-ibm-inte-3c22-deployment-885cfb758-hdrxp   2/2     Running   0          9m44s
  
  > This phase might hit the issue#3, check the issues section for more information.

- Accessing the Platform Navigator

  - Use the following command to get the URL
  
        # oc get routes -n openshift-operators
        NAME                       HOST/PORT                                                         PATH   SERVICES                                                      PORT    TERMINATION          WILDCARD
        integration-navigator-pn   integration-navigator-pn-openshift-operators.apps.s07.mcocp.com          integration-navigator-ibm-integration-platform-navigator-ui   <all>   reencrypt/Redirect   None
        
  - Add the entry to the /etc/hosts in your laptop
  
        9.115.251.187 mc187 integration-navigator-pn-openshift-operators.apps.s07.mcocp.com     
   
## 4. Installing other common services if needed

- Prepare a YAML file `operand-request.yaml` for Common UI and create it

  ```bash
  # cat <<EOF> operand-request.yaml
  apiVersion: operator.ibm.com/v1alpha1
  kind: OperandRequest
  metadata:
    name: common-service
    namespace: ibm-common-services
    labels:
      app.kubernetes.io/instance: operand-deployment-lifecycle-manager
      app.kubernetes.io/managed-by: operand-deployment-lifecycle-manager
      app.kubernetes.io/name: odlm
  spec:
    requests:
      - operands:
          - name: ibm-healthcheck-operator
          - name: ibm-licensing-operator
            bindings:
              public-api-token:
                secret: ibm-licensing-token
              public-api-upload:
                secret: ibm-licensing-upload-token
                configmap: ibm-licensing-upload-config
          - name: ibm-metering-operator
            bindings:
              public:
                secret: icp-metering-api-secret
          - name: ibm-events-operator
          - name: ibm-elastic-stack-operator
            bindings:
              public-certs:
                secret: logging-elk-certs
              public-passphrases:
                secret: logging-elk-elasticsearch-pki-secret
          - name: ibm-auditlogging-operator
            bindings:
              public:
                secret: audit-server-certs
                configmap: audit-logging-fluentd-ds-httpingest-url
        registry: common-service
        registryNamespace: ibm-common-services
  EOF
    
  # oc apply -f operand-request.yaml
  operandrequest.operator.ibm.com/common-service created
  ```

## 5. Uninstalling the IBM Cloud Pak for Integration

### 5.1 Removing the operands using Openshift web console

- From the left navigation pane of OCP web console, click `Operators` > `Installed Operators`.
- Select the namespace your operands are deployed, e.g. `openshift-operators` and `ibm-common-services` here.
- Delete the operands associated with the operator using the tabs for each Custom Resource managed by the operator.

  - From Platform Navigators -> integration-navigator instance
  - From the `OperandRequest` tab of ODLM, delete the OperandRequest `common-service`.

### 5.2 Uninstalling the operators using Openshift web console

- From the left navigation pane of OCP web console, click `Operators` > `Installed Operators`.
- Select the namespace `all projects`, uninstall below operators one by one.
  
  ```
  Operator for Apache CouchDB
  IBM DataPower Gateway
  IBM API Connect
  IBM Aspera HSTS
  IBM Cert Manager
  IBM Operator for Redis
  IBM Cloud Platform Common Services
  IBM Cloud Pak for Integration
  IBM Event Streams
  IBM Cloud Bak for Integration Asset Repository
  IBM Cloud Bak for Integration Operations Dashboard
  IBM Cloud Bak for Integration Platform Navigator
  IBM MQ
  IBM NamespaceScope Operator
  Operand Deployment Lifecycle Manager
  ```

### 5.3 Removing the remaining deployment and services if have

- Verify if have remaining deployment on previous deployed namespace

  ```bash
  # oc get deployments -n openshift-operators
  # oc get deployments -n ibm-common-services
  
  
  Examples:
  # oc get deployments -n openshift-operators
  No resources found in openshift-operators namespace.
  # oc get deployments -n ibm-common-services
  NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
  ibm-common-service-webhook   1/1     1            1           9d
  secretshare                  1/1     1            1           9d
  ```

- Delete the remaining deployments

  ```bash
  # oc delete deployments ibm-common-service-webhook -n ibm-common-services
  deployment.apps "ibm-common-service-webhook" deleted
  # oc delete deployments secretshare -n ibm-common-services
  deployment.apps "secretshare" deleted
  ```

- Verify if have remaining services on previous deployed namespace

  ```bash
  # oc get services -n openshift-operators
  # oc get services -n ibm-common-services
  
  Examples:
  # oc get services -n openshift-operators
  No resources found in openshift-operators namespace.
  # oc get services -n ibm-common-services
  NAME                         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
  ibm-common-service-webhook   ClusterIP   None         <none>        443/TCP   9d
  ```

- Delete the remaining services

  ```bash
  # oc delete services ibm-common-service-webhook -n ibm-common-services
  service "ibm-common-service-webhook" deleted
  ```

### 5.4 Remove the following configmaps and namespace

- Verify the remaining secrets and configmap

  ```bash
  # oc -n kube-system get secret | grep icp
  # oc -n kube-public get configmap | grep ibm
  
  Example:
  # oc -n kube-system get secret | grep icp
  icp-metering-api-secret                          kubernetes.io/tls                     3      12d
  icp-mongodb-admin                                Opaque                                2      9d
  icp-mongodb-client-cert                          kubernetes.io/tls                     3      9d
  icp-serviceid-apikey-secret                      Opaque                                1      9d
  
  # oc -n kube-public get configmap | grep ibm
  ibm-common-services-status   1      9d
  ```

- Delete the remaining secrets, configmap and namespace
  
  ```bash
  # oc -n kube-system delete secret icp-metering-api-secret
  # oc -n kube-system delete secret icp-mongodb-admin
  # oc -n kube-system delete secret icp-mongodb-client-cert
  # oc -n kube-system delete secret icp-serviceid-apikey-secret
  # oc -n kube-public delete configmap ibm-common-services-status
  # oc delete mutatingwebhookconfiguration ibm-common-service-webhook-configuration
  # oc delete namespace services
  ```

- Verify the remaining namespacescopes
  
  ```bash
  # oc -n ibm-common-services get namespacescopes
  
  Example:
  # oc -n ibm-common-services get namespacescopes
  NAME                 AGE
  common-service       9d
  namespacescope       9d
  nss-managedby-odlm   9d
  ```

- Delete the remaining namespacescopes

  ```bash
  # for i in `oc -n ibm-common-services get namespacescopes | grep -v NAME | awk -F " " '{print $1}'`; do oc -n ibm-common-services patch namespacescopes $i -p '{"metadata":{"finalizers":[]}}' --type=merge; done
  # oc -n ibm-common-services delete namespacescopes common-service
  # oc -n ibm-common-services delete namespacescopes namespacescope
  # oc -n ibm-common-services delete namespacescopes nss-managedby-odlm
  ```

- Delete the namespace used for CP4I

  ```bash
  # oc delete ns ibm-common-services
  ```

### 5.5 Manually delete the relevant CRDs if needed

   ```bash
   # for i in `oc get crd | grep ibm.com | grep -v csi | grep -v scale | awk -F " " '{print $1}'`; do oc delete crd $i; done
   ```

## 6. Issues I met

### Issue#1

**Openshift Console showed the operator IBM DataPower Gateway v1.2.0 as failed**

After checking the POD events, it showed below warning, but it is a <a href="https://www.ibm.com/support/knowledgecenter/en/SSGT7J_20.4/install/install_online_catalog_sources.html" target="_blank">known issue</a> and fixed by IBM DataPower Gateway v1.2.1. So this version v1.2.0 can be removed directly.
```
  Warning  FailedScheduling  14s   default-scheduler  0/6 nodes are available: 3 node(s) didn't match pod topology spread constraints (missing required label), 3 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate.
```

### Issue#2

**Can't remove IBM DataPower Gateway v1.2.0**

The operator is a deployment, if deleting it via `oc` directly, it will be started a new one by OLM.

To delete it, list the failed operator under `Installed Operators` of OpenShift web console, click action `Delete ClusterServiceVersion`, ensure the checkbox of `Delete dependent objects of this resource` selected, then click `Delete` button.

### Issue#3
**Many PODs failed to be running during deploying Platform Navigator instance**

    # oc get pod
    NAME                                                      READY   STATUS                       RESTARTS   AGE
    alertmanager-ibm-monitoring-alertmanager-0                3/3     Running                      0          12m
    auth-pap-77fc94859-q7rvc                                  2/2     Running                      0          16m
    auth-pap-77fc94859-z4cjf                                  2/2     Running                      0          16m
    auth-pdp-546f564479-hd7rb                                 1/2     CreateContainerConfigError   0          16m
    auth-pdp-546f564479-v8wpd                                 1/2     CreateContainerConfigError   0          16m
    auth-pdp-5f49dccdb9-4b96c                                 1/2     CreateContainerConfigError   0          14m
    cert-manager-cainjector-cf66985fb-pzcv9                   1/1     Running                      0          45m
    cert-manager-controller-69bf69d79b-chb8k                  1/1     Running                      0          45m
    cert-manager-webhook-798d55f5-mzd8j                       1/1     Running                      2          45m
    common-web-ui-8588fc7876-x5t5z                            1/2     CreateContainerConfigError   0          16m
    common-web-ui-8588fc7876-xt662                            1/2     CreateContainerConfigError   0          16m
    configmap-watcher-87776db9f-5vkx7                         1/1     Running                      0          45m
    default-http-backend-75c8b889cb-szhqv                     1/1     Running                      0          14m
    iam-onboarding-7rnkd                                      0/1     Init:0/5                     0          16m
    iam-policy-controller-55645f9d88-nlgq5                    1/1     Running                      0          16m
    ibm-cert-manager-operator-7ff94cb867-k8md9                1/1     Running                      0          45m
    ibm-common-service-webhook-dbd6778cb-gwmxr                1/1     Running                      0          47m
    ibm-commonui-operator-69d9fc855-wjpn8                     1/1     Running                      0          17m
    ibm-iam-operator-68db9bbdd9-k4gqf                         1/1     Running                      0          17m
    ibm-ingress-nginx-operator-79d9649689-kszv5               1/1     Running                      0          15m
    ibm-management-ingress-operator-76997c6c9c-rgvsb          1/1     Running                      0          15m
    ibm-mongodb-operator-8c88ddf4b-zdld8                      1/1     Running                      0          15m
    ibm-monitoring-collectd-7cfd6866b7-xqjlg                  2/2     Running                      0          16m
    ibm-monitoring-exporters-operator-7577f49fc8-5zns8        1/1     Running                      0          17m
    ibm-monitoring-grafana-9b759c6fc-j4595                    2/3     Running                      7          16m
    ibm-monitoring-grafana-operator-7468d6c9bc-s7ghm          1/1     Running                      0          17m
    ibm-monitoring-kube-state-68d7d87bf-pm9v8                 2/2     Running                      0          16m
    ibm-monitoring-mcm-ctl-5454c586cc-xhjjg                   1/1     Running                      0          13m
    ibm-monitoring-nodeexporter-2698h                         2/2     Running                      0          16m
    ibm-monitoring-nodeexporter-bbmx2                         2/2     Running                      0          16m
    ibm-monitoring-nodeexporter-fbw65                         2/2     Running                      0          16m
    ibm-monitoring-prometheus-operator-65b5b97f99-6jz5b       1/1     Running                      0          13m
    ibm-monitoring-prometheus-operator-ext-6f4bf48dcd-n8xlr   1/1     Running                      1          17m
    ibm-namespace-scope-operator-5659c8d556-xgm88             1/1     Running                      0          47m
    ibm-platform-api-operator-b56b6cd58-7cg2g                 1/1     Running                      0          15m
    icp-mongodb-0                                             2/2     Running                      0          15m
    icp-mongodb-1                                             2/2     Running                      0          13m
    icp-mongodb-2                                             2/2     Running                      0          8m38s
    management-ingress-67558d45cf-5ggnp                       0/1     CreateContainerConfigError   0          13m
    management-ingress-67558d45cf-dxzbs                       0/1     CreateContainerConfigError   0          13m
    nginx-ingress-controller-7bd9f6c97d-6fdls                 1/1     Running                      1          14m
    nginx-ingress-controller-7bd9f6c97d-qhrmh                 1/1     Running                      0          14m
    oidc-client-registration-vxr9q                            0/1     ContainerCreating            0          16m
    oidcclient-watcher-7bb54c84d-8p6q2                        0/1     Init:0/3                     0          16m
    operand-deployment-lifecycle-manager-8687fd5d45-rjzlx     1/1     Running                      0          46m
    platform-api-65f887b854-564dn                             2/2     Running                      0          15m
    prometheus-ibm-monitoring-prometheus-0                    3/4     CrashLoopBackOff             7          12m
    secret-watcher-9759784b7-zmlp6                            0/1     CreateContainerConfigError   0          16m
    secretshare-6b985c4fd7-zvtjc                              1/1     Running                      0          47m
    security-onboarding-bcc4c                                 0/1     Init:0/1                     0          16m

After checking POD events, the issue should be caused by some secrets delayed creation. After deleting one POD `auth-pdp` only then all failing PODs continue to be running.

### Issue#4
**During deploying Platform Navigator instance, the secret "integration-navigator-ibm-inte-3c22-oidc-client" was not generated**
```
integration-navigator-ibm-inte-3c22-deployment-885cfb758-g4t8n   0/2     CreateContainerConfigError   0          157m
integration-navigator-ibm-inte-3c22-deployment-885cfb758-t5wdc   0/2     CreateContainerConfigError   0          157m

  Warning  Failed          134m (x6 over 135m)     kubelet            Error: secret "integration-navigator-ibm-inte-3c22-oidc-client" not found
  Normal   Pulled          134m (x5 over 135m)     kubelet            Container image "cp.icr.io/cp/icp4i/icip-services:2020-11-25-1623-b398b7d7@sha256:9c97d5d91105adbfb93f8faddf02c8ac7e26e90b7cf21eb3fb3e9b580db4ac28" already present on machine
  Warning  Failed          8m29s (x584 over 139m)  kubelet            Error: secret "integration-navigator-ibm-inte-3c22-oidc-client" not found
  Normal   Pulled          3m31s (x606 over 135m)  kubelet            Container image "cp.icr.io/cp/icp4i/icip-navigator:2020-11-27-1400-bbd1e584@sha256:06f661223964e7074e603462aadf6a74b9d7b1339d7bdf8db86d76ae92c7cb9d" already present on machine
```
After further check, it should be caused by the previous uninstallation not clean, because one resource under client was remaining.
```
# oc get Client -n openshift-operators
NAME                                                       AGE
integration-navigator-ibm-inte-3c22-oidc-client            27d
```
Remove it before next new deployment then solve the issue, might empty finalizer too.

### Issue#5
**Can't delete namespace ibm-common-services**

The namespace ibm-common-services stuck in the phase "Terminating", if edit the namespace then can see the conditions showing below reason:
```
  - lastTransitionTime: "2021-02-26T02:56:41Z"
    message: 'Some resources are remaining: namespacescopes.operator.ibm.com has 2 resource instances'
    reason: SomeResourcesRemain
    status: "True"
    type: NamespaceContentRemaining
  - lastTransitionTime: "2021-02-26T02:56:41Z"
    message: 'Some content in the namespace has finalizers remaining: finalizer.nss.operator.ibm.com in 2 resource instances'
    reason: SomeFinalizersRemain
    status: "True"
    type: NamespaceFinalizersRemaining
  phase: Terminating
```

In this case, we can find two resources still remaining on namespacescopes.

```bash
# oc get namespacescopes.operator.ibm.com
NAME                 AGE
common-service       26h
nss-managedby-odlm   26h
```

To remove them, empty the finalizers then the namespace will be gone.
```
# for i in `oc -n ibm-common-services get namespacescopes | grep -v NAME | awk -F " " '{print $1}'`; do oc -n ibm-common-services patch namespacescopes $i -p '{"metadata":{"finalizers":[]}}' --type=merge; done
```

## 7. References

<a href="https://www.ibm.com/support/knowledgecenter/SSGT7J" target="_blank">IBM Cloud Pak for Integration</a> from Knowledge Center  
<a href="https://www.ibm.com/support/knowledgecenter/SSHKN6" target="_blank">IBM Cloud Platform Common Services</a> from Knowledge Center 

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.