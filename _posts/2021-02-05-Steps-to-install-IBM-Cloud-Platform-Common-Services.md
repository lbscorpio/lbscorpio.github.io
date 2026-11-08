---
title:  Steps to install IBM Cloud Platform Common Services on an OCP environment
date: 2021-02-05 18:11:50
categories:
  - Procedure
tags:
  - pd
  - cp4i
  - cs
  - ocp
---

IBM Cloud Platform Common Services provide key foundational services, e.g. logging, monitoring, common web UI and so on. Several IBM Cloud Paks will use some of those services.

## 1. Prerequisites

### 1.1 Environment Preparation

- <a href="https://www.ibm.com/support/knowledgecenter/SSHKN6/installer/3.x.x/hardware_sizing_reqs.html" target="_blank">Refer hardware sizing requirement in knowledge center</a>
- An existing OCP cluster (Red Hat OpenShift 4.4 or above), I used OCP v4.6.15 for this setup.
- For a small profile on a Linux x86_64 cluster, common services need at least 6 VPCs and 10 GB memory, minimum number of worker node is 1. For a medium profile on a Linux x86_64 cluster, common services need at least 13 VPCs and 21 GB memory, minimum number of worker node is 3.

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
   
3. Ensure a **DEFAULT** storage class, which can be used for creating storage for Common Services.

   ```bash
   # oc get storageclasses
   NAME                             PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
   sc-iscsi-fs9200-dedup            block.csi.ibm.com              Delete          Immediate              true                   8d
   sc-iscsi-fs9200-thin (default)   block.csi.ibm.com              Delete          Immediate              true                   8d
   ```
   
   Make a StorageClass as default:
   
   ```bash
   # kubectl patch storageclass <storageclass_name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
   ```

4. (Optional) For Elasticsearch of logging service, ensure that the vm.max_map_count setting is at least 262144 on all nodes.

   ```
   # sysctl -a | grep vm.max_map_count
   vm.max_map_count = 262144
   ```
   
   If the vm.max_map_count setting is not at least 262144, complete these steps to set the value to 262144:
   
   ```bash
   # cat <<EOF> tuned-cs-es.yaml
   apiVersion: tuned.openshift.io/v1
   kind: Tuned
   metadata:
    name: common-services-es
    namespace: openshift-cluster-node-tuning-operator
   spec:
    profile:
    - data: |
        [sysctl]
        vm.max_map_count=262144
      name: common-services-es
    recommend:
    - priority: 10
      profile: common-services-es
   EOF
   
   # oc create -f tuned-cs-es.yaml
   ```
   
## 2. Installing the IBM Cloud Platform Common Services operator

### 2.1 Create the IBM Cloud Platform Common Services catalog source.

- Prepare a YAML file `opencloud-operators.yaml` and create it.

   ```bash
   # cat <<EOF> opencloud-operators.yaml
   apiVersion: operators.coreos.com/v1alpha1
   kind: CatalogSource
   metadata:
     name: opencloud-operators
     namespace: openshift-marketplace
   spec:
     displayName: IBMCS Operators
     publisher: IBM
     sourceType: grpc
     image: docker.io/ibmcom/ibm-common-service-catalog:latest
     updateStrategy:
       registryPoll:
         interval: 45m
   EOF
   
   # oc create -f opencloud-operators.yaml
   ```
   
- Verify that the source container is running:
   
   ```bash
   # oc -n openshift-marketplace get pod | grep opencloud-operators
   opencloud-operators-65ncp                                         1/1     Running     0          35s
   ```

### 2.2 Install the IBM Cloud Platform Common Services operator
   
- From the navigation pane of OCP web console, click `Operators` > `OperatorHub`.
- In the _All Items_ field, enter `IBM Cloud Platform Common Services`(for v3.6.x) or `IBM Cloud Pak foundational services`(for v3.7.x).
- Click the `IBM Cloud Platform Common Services` or `IBM Cloud Pak foundational services` tile, then click `Install` button.
- Leave all options as default, click `Install` button.

### 2.3 Setting the hardware profile to small, default is medium.

- From the navigation pane of OCP web console, click `Home` > `Search`.
- From the Project drop-down list, select `ibm-common-services`.
- From the Resources drop-down list, select `CommonService`.
- Click the `common-service` resource, select the YAML tab.
- Update the `spec.size` parameter to small.
  ```yaml
  ...
  spec:
    size: small
  ```
   
## 3. Installing common services in your cluster

- Prepare a YAML file `operand-request.yaml` for Common UI as example here and create it.
    
  > Must read the <a href="https://www.ibm.com/support/knowledgecenter/SSHKN6/installer/3.x.x/disable_and_enable_services.html" target="_blank">Common service dependencies</a> before creating the YAML file.
    
  ```bash
  # cat <<EOF> operand-request.yaml
  apiVersion: operator.ibm.com/v1alpha1
  kind: OperandRequest
  metadata:
    name: common-service
    namespace: ibm-common-services
  spec:
    requests:
      - operands:
          - name: ibm-cert-manager-operator
          - name: ibm-mongodb-operator
          - name: ibm-iam-operator
            bindings:
              public-oidc-creds:
                secret: platform-oidc-credentials
              public-auth-creds:
                secret: platform-auth-idp-credentials
              public-auth-cert:
                secret: platform-auth-secret
              public-cam-secret:
                secret: oauth-client-secret
              public-cam-map:
                configmap: oauth-client-map
              public-auth-config:
                configmap: platform-auth-idp
          - name: ibm-management-ingress-operator
            bindings:
              public:
                secret: ibmcloud-cluster-ca-cert
                configmap: ibmcloud-cluster-info
          - name: ibm-commonui-operator
          - name: ibm-ingress-nginx-operator
          - name: ibm-platform-api-operator
        registry: common-service
        registryNamespace: ibm-common-services
  EOF
    
  # oc apply -f operand-request.yaml
  operandrequest.operator.ibm.com/common-service created
  ```
    
- Verifying the installation
    
  ```bash
  # oc get pods -n ibm-common-services
  NAME                                                    READY   STATUS      RESTARTS   AGE
  auth-idp-79785b8bcc-z8k2d                               4/4     Running     0          11m
  auth-pap-678ddc74cd-sprwr                               2/2     Running     0          16h
  auth-pdp-8c75fb464-2zr5t                                2/2     Running     0          10m
  cert-manager-cainjector-7c5d667b7-d9vsx                 1/1     Running     8          16h
  cert-manager-controller-595cdc44dd-ql25n                1/1     Running     0          16h
  cert-manager-webhook-798d55f5-8pdhv                     1/1     Running     0          16h
  common-web-ui-84db654d9d-q7k55                          2/2     Running     4          16h
  configmap-watcher-587667b6dd-55dvz                      1/1     Running     0          16h
  default-http-backend-8659c9f69b-fxs77                   1/1     Running     0          16h
  iam-onboarding-2k8mp                                    0/1     Completed   0          8m7s
  iam-policy-controller-c994c58cf-8g962                   1/1     Running     0          16h
  ibm-cert-manager-operator-7ff94cb867-hrt4q              1/1     Running     0          16h
  ibm-common-service-webhook-dbd6778cb-4c4xp              1/1     Running     0          5d20h
  ibm-commonui-operator-69d9fc855-lmpjg                   1/1     Running     0          16h
  ibm-iam-operator-68db9bbdd9-nmv8z                       1/1     Running     0          16h
  ibm-ingress-nginx-operator-79d9649689-7ztlw             1/1     Running     0          16h
  ibm-management-ingress-operator-76997c6c9c-tkf6z        1/1     Running     0          16h
  ibm-mongodb-operator-8c88ddf4b-fjmd8                    1/1     Running     0          16h
  ibm-namespace-scope-operator-5659c8d556-hqrrp           1/1     Running     0          5d20h
  ibm-platform-api-operator-b56b6cd58-x9zzw               1/1     Running     4          16h
  icp-mongodb-0                                           2/2     Running     0          7m11s
  management-ingress-7d4c45b444-64xww                     1/1     Running     0          16h
  nginx-ingress-controller-6b49ffdbb5-v7bq6               1/1     Running     0          16h
  oidc-client-registration-csppq                          0/1     Completed   0          16h
  oidcclient-watcher-6758986b95-mtpqq                     1/1     Running     0          7m11s
  operand-deployment-lifecycle-manager-8687fd5d45-7gm46   1/1     Running     0          5d20h
  platform-api-75b4ff5f66-w7xrw                           2/2     Running     0          16h
  secret-watcher-64b4cf87fc-pk62c                         1/1     Running     0          6m2s
  secretshare-6b985c4fd7-f22w5                            1/1     Running     0          5d20h
  security-onboarding-69fbj                               0/1     Completed   0          5m55s
    
  # oc -n ibm-common-services get csv
  NAME                                          DISPLAY                                VERSION   REPLACES                                      PHASE
  ibm-cert-manager-operator.v3.8.1              IBM Cert Manager                       3.8.1     ibm-cert-manager-operator.v3.8.0              Succeeded
  ibm-common-service-operator.v3.6.3            IBM Cloud Platform Common Services     3.6.3     ibm-common-service-operator.v3.6.2            Succeeded
  ibm-commonui-operator.v1.4.1                  Ibm Common UI                          1.4.1     ibm-commonui-operator.v1.4.0                  Succeeded
  ibm-iam-operator.v3.8.2                       IBM IAM Operator                       3.8.2     ibm-iam-operator.v3.8.1                       Succeeded
  ibm-ingress-nginx-operator.v1.4.1             IBM Ingress Nginx Operator             1.4.1     ibm-ingress-nginx-operator.v1.4.0             Succeeded
  ibm-management-ingress-operator.v1.4.3        Management Ingress Operator            1.4.3     ibm-management-ingress-operator.v1.4.2        Succeeded
  ibm-mongodb-operator.v1.2.1                   IBM MongoDB Operator                   1.2.1     ibm-mongodb-operator.v1.2.0                   Succeeded
  ibm-namespace-scope-operator.v1.0.1           IBM NamespaceScope Operator            1.0.1     ibm-namespace-scope-operator.v1.0.0           Succeeded
  ibm-platform-api-operator.v3.8.1              IBM Platform API                       3.8.1     ibm-platform-api-operator.v3.8.0              Succeeded
  operand-deployment-lifecycle-manager.v1.4.2   Operand Deployment Lifecycle Manager   1.4.2     operand-deployment-lifecycle-manager.v1.4.1   Succeeded
    ```
    
- Accessing the console

  - Use the following command to get the console URL
   
        # oc get route -n ibm-common-services cp-console -o jsonpath=‘{.spec.host}’
       ‘cp-console.apps.s07.mcocp.com’
       
  - Based on the above example output, the console URL would be https://cp-console.apps.s07.mcocp.com:443.

  - Add the entry to the /etc/hosts in your laptop
   
        9.115.251.187 mc187 cp-console.apps.s07.mcocp.com
   
  - Log in to `IBM Cloud Pak Administration Hub` with above console URL, select authentication type `OpenShift authentication`, then you can use kubeadmin to login it.
    
## 4. Uninstalling common services and the relevant CRDs

- <a href="https://www.ibm.com/support/knowledgecenter/SSHKN6/installer/3.x.x/uninstallation.html" target="_blank">Refer uninstallation in Knowledge Center</a>
- Uninstalling all services
   
  1) Uninstall the `IBM Cloud Platform Common Services` operator
     
     - From the navigation pane of OCP web console, click `Operators` > `Installed Operators`
     - Switch to the project `ibm-common-services`, locate the `IBM Cloud Platform Common Services` operator.
     - Select the action `Uninstall Operator`
  
  2) Delete the operator APIs from the `ibm-common-services` namespace.
  
     - Delete the **OperandRequest** instances
     
       - Click `Operators` > `Installed Operators` > `Operand Deployment Lifecycle Manager` > `OperandRequest`
       - Delete all the `OperandRequest` instances that you created
       
             common-service
             ibm-iam-request
             ibm-mongodb-request
             management-ingress
             platform-api-request
             ibm-commonui-request
     
       - Wait for the `OperandRequest` instances to be deleted
       
     - Delete the **OperandConfig** instances
     
       - Click `Operators` > `Installed Operators` > `Operand Deployment Lifecycle Manager` > `OperandConfig`
       - Delete all the `OperandConfig` instances (e.g. common-service here)
       
     - Delete the **OperandRegistry** instances
       - Click `Operators` > `Installed Operators` > `Operand Deployment Lifecycle Manager` > `OperandRegistry`
       - Delete all the OperandRegistry instances (e.g. common-service here)
       
     - Delete the **NamespaceScope** instances
     
       - Click `Operators` > `Installed Operators` > `IBM NamespaceScope Operator` > `Namespace Scope`
       - Delete all the `NamespaceScope` instances (e.g. common-service and nss-managedby-odlm here)
  
  3) Uninstall the `Operand Deployment Lifecycle Manager` operator
  
     - From the navigation pane of OCP web console, click `Operators` > `Installed Operators`
     - Switch to the project `ibm-common-services`, locate the `Operand Deployment Lifecycle Manager operator` operator.
     - Select the action `Uninstall Operator`
    
  You also can run the following script to remove all services components, it will do the same actions from above three steps.
  
  ```bash
  # bash <(curl -s https://raw.githubusercontent.com/IBM/ibm-common-service-operator/master/common/scripts/force-uninstall.sh)
  ```
  
  4) Remove the following configmaps and namespace
  
     ```bash
     # oc -n kube-system delete secret icp-mongodb-admin
     # oc -n kube-system delete secret icp-mongodb-client-cert
     # oc -n kube-system delete secret icp-serviceid-apikey-secret
     # oc -n kube-public delete configmap ibm-common-services-status
     # oc delete mutatingwebhookconfiguration ibm-common-service-webhook-configuration
     # oc delete namespace services
     ```

- Manually delete the CRDs that you do not want
  
  1) Get the CRDs
     
         # oc get crd | grep ibm.com | grep -v csi | grep -v scale
         authentications.operator.ibm.com                            2021-02-12T12:47:25Z
         certmanagers.operator.ibm.com                               2021-02-12T12:47:33Z
         clients.oidc.security.ibm.com                               2021-02-12T12:47:25Z
         commonservices.operator.ibm.com                             2021-02-07T08:37:21Z
         commonwebuis.operators.ibm.com                              2021-02-12T12:47:30Z
         iampolicies.iam.policies.ibm.com                            2021-02-12T12:47:24Z
         legacyheaders.operators.ibm.com                             2021-02-12T12:47:30Z
         managementingresses.operator.ibm.com                        2021-02-12T12:47:36Z
         mongodbs.operator.ibm.com                                   2021-02-12T12:47:44Z
         namespacescopes.operator.ibm.com                            2021-02-07T08:38:02Z
         navconfigurations.foundation.ibm.com                        2021-02-12T12:47:31Z
         nginxingresses.operator.ibm.com                             2021-02-12T12:47:41Z
         oidcclientwatchers.operator.ibm.com                         2021-02-12T12:47:24Z
         operandbindinfos.operator.ibm.com                           2021-02-07T08:38:48Z
         operandconfigs.operator.ibm.com                             2021-02-07T08:38:48Z
         operandregistries.operator.ibm.com                          2021-02-07T08:38:48Z
         operandrequests.operator.ibm.com                            2021-02-07T08:38:48Z
         paps.operator.ibm.com                                       2021-02-12T12:47:24Z
         platformapis.operator.ibm.com                               2021-02-12T12:47:46Z
         podpresets.operator.ibm.com                                 2021-02-07T08:38:12Z
         policycontrollers.operator.ibm.com                          2021-02-12T12:47:25Z
         policydecisions.operator.ibm.com                            2021-02-12T12:47:25Z
         secretshares.ibmcpcs.ibm.com                                2021-02-07T08:38:18Z
         secretwatchers.operator.ibm.com                             2021-02-12T12:47:25Z
         securityonboardings.operator.ibm.com                        2021-02-12T12:47:25Z
         switcheritems.operators.ibm.com                             2021-02-12T12:47:30Z
  
  2) Delete the CRDs that you do not want
  
         # oc delete crd <CRD1>, <CRD2>, ..., <CRDn>
         
         One-line command for this example ONLY:
         # for i in `oc get crd | grep ibm.com | grep -v csi | grep -v scale | awk -F " " '{print $1}'`; do oc delete crd $i; done
         
         If some crd deletion stuck, empty the finalizers.
         # oc patch crd <CRDx> -p '{"metadata":{"finalizers":[]}}' --type=merge
         
         One-line command for this example ONLY:
         # for i in `oc get crd | grep ibm.com | grep -v csi | grep -v scale | awk -F " " '{print $1}'`; do oc patch crd $i -p '{"metadata":{"finalizers":[]}}' --type=merge; done
         
- Manually delete the namespace `ibm-common-services` if still there

      # oc delete ns ibm-common-services
      
## 5. References

<a href="https://www.ibm.com/support/knowledgecenter/SSHKN6/kc_welcome_cs.html" target="_blank">IBM Cloud Platform Common Services</a> from Knowledge Center  
<a href="https://www.ibm.com/support/knowledgecenter/SSGT7J" target="_blank">IBM Cloud Pak for Integration</a> from Knowledge Center  
<a href="https://github.ibm.com/IBMPrivateCloud/BedrockServices" target="_blank">Bedrock Services</a> from GitHub 

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.