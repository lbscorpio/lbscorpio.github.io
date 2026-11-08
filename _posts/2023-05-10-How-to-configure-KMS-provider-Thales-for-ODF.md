---
title:  How to configure KMS provider Thales for ODF
date: 2023-05-10 16:26:28
categories:
  - Procedure
tags:
  - fusion
  - ocp
  - odf
  - kms
  - thales
---

OpenShift Data Foundation support cluster-wide encryption during first deployment, you can connect to an external key management service. Starting from OCP 4.12, ODF support third-part KMS provider Thales CipherTrust Manager(using KMIP) too.

This article will guide you how to register a new KMIP Client on Thales CipherTrust Manager for ODF deployment.

## Prerequisites

- An existing OpenShift cluster with 4.12, have installed ODF operator.
- Have deployed Thales <a href="https://thalesdocs.com/ctp/cm/latest/get_started/deployment/index.html" target="_blank">CipherTrust Manager</a> with a valid license, the above OpenShift cluster can access it. Below instructions based on CipherTrust Manager k170v version 2.9.0+7637.

## Setting up the environment

### Install the CLI Toolkit `ksctl`

- Download and unzip the ksctl_images.zip file
    
  1. Enter the IP address of your CipherTrust Manager system in your browser.
    
  2. If you are logged into CipherTrust Manager, select the **API** link at the top right. If you are logged out, select **API & CLI Documentation** at the bottom right.
    
  3. Select the **CLI Guide** page and click on Download CLI button on the upper right, or you can use below download URL:
    
     ```
     https://<IP_of_CipherTrustManager>/downloads/ksctl_images.zip
     ```
    
  4. Unzip the **ksctl_images.zip** file.
    
  5. Rename the file `ksctl-darwin-amd64`(for MacOS) or `ksctl-linux-amd64`(for Linux) to `ksctl`, then move it to the directory `/usr/local/bin/`.

### Create a ksctl configuration file 

- Create a `.ksctl` directory in your **$HOME** directory:

  ```
  $ cd ~
  $ mkdir .ksctl
  ```

- Create a `config.yaml` file with the following items:

  ```
  KSCTL_USERNAME: <user_in_ciphertrust_manager>
  KSCTL_PASSWORD: <your_user_password>
  KSCTL_URL: https://<ciphertrust_manager_ipaddress>
  KSCTL_NOSSLVERIFY: true
  ```

### Run ksctl commands

- Run `users` command to verify communication with CipherTrust Manager.

  ```
  $ ksctl users list
  ```
  
- Run help command for more information

  ```
  $ ksctl --help
  $ ksctl users --help
  ```

## To Register a new KMIP Client

### Create a json file for specifying properties to create a KMIP client profile

- Create a json file `profile.json` with below parameters

  ```json
  {
    "cert_user_field": "CN",
    "csr_cn": "user1",
    "csr_org_name": "IBM",
    "csr_org_unit": "CSL",
    "csr_email": "xxx@ibm.com",
    "csr_city": "Shanghai",
    "csr_state": "Shanghai",
    "csr_country": "CN"
  }
  ```

  > Note: The above `user1` should be the exact username existing in CipherTrust Manager.

- Create the KMIP profile `isf-odf` with above json file

  ```
  $ ksctl kmip createprofile -p isf-odf -c profile.json
  ```

### Create a registration token using the previous client profile

Using CipherTrust Manager GUI to create a registration token, refer to <a href="https://thalesdocs.com/ctp/cm/2.1/admin/cm_admin/client-management/index.html#creating-a-registration-token" target="_blank">Creating a Registration Token‌‌</a> for more information.

- Log on to the CipherTrust Manager GUI as administrator.

- In the left pane, click `Access Management` -> `Registration Tokens`.

- On the right, click `Add Registration Token`.

- Click `Begin` to start token creation.

- Specify a `Name Prefix`, click `Next` and select the local CA.

- Click `Create Token`, then click `Copy` next to the token.

- Click `Add Token`.

  > Note: Using `ksctl` to create the registration token might be failed, it can't find the previous client profile in my testing.

### Register the new KMIP client using the registration token generated in the previous step

- If client registration is through local CA, you will have the signed client certificate and the client private key files.

  ```
  $ ksctl kmip register -n isf-odf -t <previous_registration_token> --kmipCertOutFile ClientCert.pem --kmipKeyOutFile ClientKey.pem
  ```

### Download the local CA certificate from GUI

- Log in to the CipherTrust Manager console as an administrator with Certificate Authorities access controls.

- Navigate to `CA` > `Local Certificate` Authorities and click the download button to download the specific local CA.

> Note: Refer to <a href="https://thalesdocs.com/ctp/con/cakm/cakm-oracle-tde/latest/admin/tde_advanced_topics/tde-setting_up_ssltls/index.html#downloading-a-local-ca-certificate" target="_blank">Downloading a Local CA Certificate‌‌</a> for more information.

## References

<a href="https://thalesdocs.com/ctp/cm/2.9/reference/kmip-ref/index.html" target="_blank">KMIP Reference‌‌</a> from CipherTrust Manager  
<a href="https://thalesdocs.com/ctp/cm/2.9/admin/cm_admin/index.html" target="_blank">CipherTrust Manager Administration‌‌</a>

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.