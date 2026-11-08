---
title:  How to use ocs-ci framework for testing
date: 2021-06-03 10:05:18
categories:
  - Setup
tags:
  - pd
  - ocs-ci
  - python
  - pytest
---

OCS CI is a framework to test Red Hat OCS features and functionality using AWS and other supported platforms. The framework is based on pytest.

## Prerequisites

- An existing OpenShift cluster, might install OCS and create a storagecluster.
- Don't configure htpass-secret oauth if want to run UI testing, due to it won't work on such login page.
- Don't setup proxy on the client which will run `run-ci` command.
- Python >= 3.7 on the client which will run `run-ci` command.
- The `oc` client binary is installed on the client which will run `run-ci` command, and can admin access to the OCP cluster.

## Setup a python virtual environment

- Clone the `ocs-ci` repository on your client
  
      $ git clone git@github.com:red-hat-storage/ocs-ci.git 

- Setup a python virtual environment

      $ cd ocs-ci
      $ python3 -m venv venv
      (venv) $ source venv/bin/activate

- Upgrade pip and install requirements

      (venv) $ pip install --upgrade pip setuptools
      (venv) $ pip install -r requirements.txt

  > Notes: If your macOS upgraded to Big Sur(v11.x) from older versions, please refer the known issues from <a href="https://pages.github.ibm.com/lbosh/myblog/troubleshoot/2021/05/15/Two-issues-when-setup-python-virtual-environment-for-ocs-ci/" target="_blank">here</a>.
       
- Install pre-config
 
      (venv) $ pip install -r requirements-dev.txt
      (venv) $ pre-commit install --hook-type pre-commit --hook-type commit-msg

## Required configuration

- Cluster directory of this OCP/OCS cluster on your machine, with all files as produced by `openshift-install` tool. You can copy it to local `ocs-ci` directory, I renamed it to `cluster-mc187` in my case. This is required by the parameter `--cluster-path` of command `run-ci`.
- Configure your ocs-ci.yaml, which is required by `–ocsci-conf` parameter of command `run-ci`. I created a file `conf/upi_myconfig.yaml`, its content as below:

      ---
      RUN:
        kubeconfig_location: "auth/config"

      DEPLOYMENT:
        force_download_installer: False
        force_download_client: False
        skip_download_client: True
        live_deployment: True
        live_content_source: "redhat-operators"

      ENV_DATA:
        cluster_name: s07
        platform: 'baremetal'
        deployment_type: 'upi'
        skip_ocp_deployment: true
        skip_ocs_deployment: true
 
## Run the testing

- If you have installed OCS and created a storagecluster, then can try to run below test case.

      (venv) $ run-ci -k TestCreateSCSameName --cluster-path cluster-mc187 --ocsci-conf conf/upi_myconfig.yaml tests/manage/storageclass
      Failed to fetch auth.yaml from ocs-ci-data
      2021-05-11 14:00:38,370 - MainThread - ocs_ci.framework.pytest_customization.ocscilib - INFO - Dump of the consolidated config file is located here: /tmp/run-1620712828-config.yaml
      2021-05-11 14:00:38,883 - MainThread - ocs_ci.framework.pytest_customization.ocscilib - INFO - Skipping version collection because we skipped the OCS deployment
      ============================================================================= test session starts ==============================================================================
      platform darwin -- Python 3.9.4, pytest-5.3.5, py-1.10.0, pluggy-0.13.1
      rootdir: /Users/lbosh/Projects/PuDong/ocs-ci, inifile: pytest.ini
      plugins: marker-bugzilla-0.9.4, metadata-1.11.0, html-2.1.1, ordering-0.6, logger-0.5.1, reportportal-1.10.3
      collected 21 items / 20 deselected / 1 selected
      
      tests/manage/storageclass/test_create_storageclass_with_same_name.py::TestCreateSCSameName::test_create_storageclass_with_same_name
      -------------------------------------------------------------------------------- live log setup --------------------------------------------------------------------------------
      14:00:38 - MainThread - ocs_ci.utility.utils - INFO - Retrieving the authentication config dictionary
      14:00:38 - MainThread - tests.conftest - INFO - All logs located at /tmp/ocs-ci-logs-1620712828
      14:00:41 - MainThread - ocs_ci.utility.utils - INFO - Executing command: /Users/lbosh/Projects/PuDong/ocs-ci/bin/oc version --client -o json
      14:00:41 - MainThread - ocs_ci.utility.utils - INFO - OpenShift Client version: None
      14:00:41 - MainThread - tests.conftest - INFO - Skipping version reporting since OCS deployment is skipped.
      14:00:41 - MainThread - ocs_ci.utility.utils - INFO - testrun_name: OCS4-7-Downstream-OCP4-7-BAREMETAL-UPI-1AZ-RHCOS-3M-3W-
      14:00:41 - Dummy-1 - ocs_ci.utility.utils - INFO - Executing command: oc get StorageClass  -A -o yaml
      14:00:41 - Dummy-2 - ocs_ci.utility.utils - INFO - Executing command: oc get CephFileSystem  -A -o yaml
      14:00:41 - Dummy-3 - ocs_ci.utility.utils - INFO - Executing command: oc get CephBlockPool  -A -o yaml
      14:00:41 - Dummy-4 - ocs_ci.utility.utils - INFO - Executing command: oc get Pod  -A -o yaml
      14:00:41 - Dummy-5 - ocs_ci.utility.utils - INFO - Executing command: oc get PersistentVolume  -A -o yaml
      14:00:41 - Dummy-6 - ocs_ci.utility.utils - INFO - Executing command: oc get PersistentVolumeClaim  -A -o yaml
      14:00:41 - Dummy-7 - ocs_ci.utility.utils - INFO - Executing command: oc get Namespace  -A -o yaml
      14:00:57 - MainThread - tests.conftest - INFO - Checking for Ceph Health OK
      14:00:57 - MainThread - ocs_ci.utility.utils - INFO - Executing command: oc wait --for condition=ready pod -l app=rook-ceph-tools -n openshift-storage --timeout=120s
      14:00:57 - MainThread - ocs_ci.utility.utils - INFO - Executing command: oc -n openshift-storage get pod -l 'app=rook-ceph-tools' -o jsonpath='{.items[0].metadata.name}'
      14:00:58 - MainThread - ocs_ci.utility.utils - INFO - Executing command: oc -n openshift-storage exec rook-ceph-tools-7ddd664854-hln4k -- ceph health
      14:00:59 - MainThread - ocs_ci.utility.utils - INFO - Ceph cluster health is HEALTH_OK.
      14:00:59 - MainThread - tests.conftest - INFO - Ceph health check passed at setup
      14:00:59 - MainThread - ocs_ci.utility.utils - INFO - Executing command: ['oc', 'login', '-u', 'kubeadmin', '-p', '*****']
      14:01:01 - MainThread - ocs_ci.utility.utils - INFO - Executing command: oc -n openshift-monitoring whoami --show-token
      14:01:02 - MainThread - ocs_ci.utility.utils - INFO - Executing command: oc -n openshift-monitoring get Route prometheus-k8s -n openshift-monitoring -o yaml
      14:01:02 - MainThread - tests.conftest - ERROR - There was a problem with connecting to Prometheus
      Traceback (most recent call last):
        File "/Users/lbosh/Projects/PuDong/ocs-ci/tests/conftest.py", line 2895, in log_alerts
          prometheus = PrometheusAPI()
        File "/Users/lbosh/Projects/PuDong/ocs-ci/ocs_ci/utility/prometheus.py", line 320, in __init__
          self.generate_cert()
        File "/Users/lbosh/Projects/PuDong/ocs-ci/ocs_ci/utility/prometheus.py", line 352, in generate_cert
          kubeconfig["clusters"][0]["cluster"]["certificate-authority-data"]
      KeyError: 'certificate-authority-data'
      14:01:02 - MainThread - tests.conftest - ERROR - There was a problem with collecting alerts for analysis
      Traceback (most recent call last):
        File "/Users/lbosh/Projects/PuDong/ocs-ci/tests/conftest.py", line 2901, in _collect_alerts
          alerts_response = prometheus.get(
      AttributeError: 'NoneType' object has no attribute 'get'
      -------------------------------------------------------------------------------- live log call ---------------------------------------------------------------------------------
      14:01:02 - MainThread - ocs_ci.ocs.resources.ocs - INFO - Adding StorageClass with name ocs-322-sc
      14:01:02 - MainThread - ocs_ci.utility.templating - INFO - allowVolumeExpansion: true
      apiVersion: storage.k8s.io/v1
      kind: StorageClass
      metadata:
        name: ocs-322-sc
      parameters:
        clusterID: openshift-storage
        csi.storage.k8s.io/controller-expand-secret-name: '*****'
        csi.storage.k8s.io/controller-expand-secret-namespace: '*****'
        csi.storage.k8s.io/node-stage-secret-name: '*****'
        csi.storage.k8s.io/node-stage-secret-namespace: '*****'
        csi.storage.k8s.io/provisioner-secret-name: '*****'
        csi.storage.k8s.io/provisioner-secret-namespace: '*****'
        imageFeatures: layering
        imageFormat: '2'
        pool: rbd
      provisioner: openshift-storage.rbd.csi.ceph.com
      reclaimPolicy: Delete
      
      14:01:02 - MainThread - ocs_ci.utility.utils - INFO - Executing command: oc create -f /var/folders/lx/544p3srs67bfwp_476n_dwb00000gn/T/StorageClassmps2jui2 -o yaml
      14:01:02 - MainThread - ocs_ci.utility.utils - INFO - Executing command: oc get StorageClass ocs-322-sc -o yaml
      14:01:03 - MainThread - tests.manage.storageclass.test_create_storageclass_with_same_name - INFO - Storage class: ocs-322-sc created successfully !
      14:01:03 - MainThread - tests.manage.storageclass.test_create_storageclass_with_same_name - INFO - Attempting to create another storageclass with duplicate name ocs-322-sc
      14:01:03 - MainThread - ocs_ci.ocs.resources.ocs - INFO - Adding StorageClass with name ocs-322-sc
      14:01:03 - MainThread - ocs_ci.utility.templating - INFO - allowVolumeExpansion: true
      apiVersion: storage.k8s.io/v1
      kind: StorageClass
      metadata:
        name: ocs-322-sc
      parameters:
        clusterID: openshift-storage
        csi.storage.k8s.io/controller-expand-secret-name: '*****'
        csi.storage.k8s.io/controller-expand-secret-namespace: '*****'
        csi.storage.k8s.io/node-stage-secret-name: '*****'
        csi.storage.k8s.io/node-stage-secret-namespace: '*****'
        csi.storage.k8s.io/provisioner-secret-name: '*****'
        csi.storage.k8s.io/provisioner-secret-namespace: '*****'
        imageFeatures: layering
        imageFormat: '2'
        pool: rbd
      provisioner: openshift-storage.rbd.csi.ceph.com
      reclaimPolicy: Delete
      
      14:01:03 - MainThread - ocs_ci.utility.utils - INFO - Executing command: oc create -f /var/folders/lx/544p3srs67bfwp_476n_dwb00000gn/T/StorageClassgnjyin6c -o yaml
      14:01:03 - MainThread - ocs_ci.utility.utils - WARNING - Command stderr: Error from server (AlreadyExists): error when creating "/var/folders/lx/544p3srs67bfwp_476n_dwb00000gn/T/StorageClassgnjyin6c": storageclasses.storage.k8s.io "ocs-322-sc" already exists
      
      14:01:03 - MainThread - tests.manage.storageclass.test_create_storageclass_with_same_name - INFO - Cannot create two StorageClasses with same name ! Error message:
      Error during execution of command: oc create -f /var/folders/lx/544p3srs67bfwp_476n_dwb00000gn/T/StorageClassgnjyin6c -o yaml.
      Error is Error from server (AlreadyExists): error when creating "/var/folders/lx/544p3srs67bfwp_476n_dwb00000gn/T/StorageClassgnjyin6c": storageclasses.storage.k8s.io "ocs-322-sc" already exists
      
      PASSED                                                                                                                                                                   [100%]
      ------------------------------------------------------------------------------ live log teardown -------------------------------------------------------------------------------
      14:01:03 - MainThread - tests.manage.storageclass.test_create_storageclass_with_same_name - INFO - Deleting created storage class: ocs-322-sc
      14:01:03 - MainThread - ocs_ci.utility.utils - INFO - Executing command: oc delete StorageClass ocs-322-sc
      14:01:04 - MainThread - tests.manage.storageclass.test_create_storageclass_with_same_name - INFO - Storage class: ocs-322-sc deleted successfully
      14:01:04 - Dummy-4 - ocs_ci.utility.utils - INFO - Executing command: oc get Pod  -A -o yaml
      14:01:04 - Dummy-6 - ocs_ci.utility.utils - INFO - Executing command: oc get StorageClass  -A -o yaml
      14:01:04 - Dummy-3 - ocs_ci.utility.utils - INFO - Executing command: oc get PersistentVolumeClaim  -A -o yaml
      14:01:04 - Dummy-2 - ocs_ci.utility.utils - INFO - Executing command: oc get CephFileSystem  -A -o yaml
      14:01:04 - Dummy-1 - ocs_ci.utility.utils - INFO - Executing command: oc get CephBlockPool  -A -o yaml
      14:01:04 - Dummy-5 - ocs_ci.utility.utils - INFO - Executing command: oc get PersistentVolume  -A -o yaml
      14:01:04 - Dummy-7 - ocs_ci.utility.utils - INFO - Executing command: oc get Namespace  -A -o yaml
      
      
      ====================================================================== 1 passed, 20 deselected in 40.68s =======================================================================

## Some tips on writing tests

- Some official usage can check <a href="https://ocs-ci.readthedocs.io/en/latest/writing_tests.html" target="_blank">here</a>  
- For ui testing, can call the function `take_screenshot()` for debugging, which will save screenshot to `/tmp`. 

## References

<a href="https://ocs-ci.readthedocs.io/en/latest/getting_started.html" target="_blank">Getting Started for `ocs-ci`</a>  
<a href="https://ocs-ci.readthedocs.io/en/latest/usecases/running_tests_on_cluster_not_deployed_by_ocsci.html" target="_blank">How to execute tests on a cluster not deployed by ocs-ci?</a>  

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.