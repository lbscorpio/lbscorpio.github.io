---
title:  How to create certificate chain with OpenSSL
date: 2022-01-28 10:32:21
categories:
  - Procedure
tags:
  - fusion
  - linux
  - openssl
  - certificate
---

Some testing cases might require certificate chain, <a href="https://daymvs1.pok.ibm.com/ibmca/certificateProfiles.do?lang=en" target="_blank">IBM Internal CA</a> can sign CSR for internal usage, but it will add two more fields(UID and mail) to the subject name of signed certificate, but some application(e.g. SGKLM) will check subject name during importing the signed certificate.

This article will guide you how to create root and intermediate certificates, then sign CSR with OpenSSL.

## Prerequisites

- RHEL/CentOS 7/8
- OpenSSL >= 1.1.1

## Create Root CA directory structure

- Create directories to store the certificates
  
      # mkdir /root/tls
      # cd /root/tls
      [~/tls]# mkdir certs private

- Create files that our CA infrastructure will need

      [~/tls]# echo 01 > serial
      [~/tls]# touch index.txt

## Configure openssl.cnf for Root CA Certificate

- Copy the default configuration file to the Root CA directory
  
      # cp /etc/pki/tls/openssl.cnf /root/tls/.

- Update below contents(the default ones not list here) and save

      [~/tls]# vi openssl.cnf
      [ CA_default ]
      dir             = /root/tls        
      new_certs_dir   = $dir/certs
      certificate     = $dir/certs/cacert.pem

      [ v3_ca ]
      basicConstraints = critical,CA:true

      [ v3_intermediate_ca ]
      subjectKeyIdentifier = hash
      authorityKeyIdentifier = keyid:always,issuer
      basicConstraints = critical,CA:true
      keyUsage = critical, digitalSignature, cRLSign, keyCertSign

      [ v3_intermediate_ca_n ]
      subjectKeyIdentifier = hash
      authorityKeyIdentifier = keyid:always,issuer
      basicConstraints = critical,CA:true,pathlen:0
      keyUsage = critical, digitalSignature, cRLSign, keyCertSign

## Generate Root CA Private Key

- Create root CA key using 4096 bits and 3DES encryption

      [~/tls]# openssl genrsa -des3 -out private/cakey.pem 4096

## Create Root CA Certificate

- Use above root CA key to create a Root CA certificate

      [~/tls]# openssl req -new -x509 -days 3650 -config openssl.cnf -extensions v3_ca -key private/cakey.pem -out certs/cacert.pem

**NOTE**: The Common Name (CN) of the CA and the Server certificates must NOT match.

## Create Intermediate CA directory structure

- Create directories to store the certificates

      # mkdir /root/tls/intermediate
      # cd /root/tls/intermediate
      [intermediate]# mkdir certs csr private

- Create files that our CA infrastructure will need

      [intermediate]# touch index.txt
      [intermediate]# echo 01 > serial

## Configure openssl.cnf for Intermediate CA Certificate

- Copy the file `openssl.cnf` used for our Root CA Certificate

      # cp /root/tls/openssl.cnf /root/tls/intermediate/.

- Update below options for Intermediate CA Certificate

      [intermediate]# vi openssl.cnf
      [ CA_default ]
      dir             = /root/tls/intermediate
      certificate     = $dir/certs/intermediate.cacert.pem
      private_key     = $dir/private/intermediate.cakey.pem
      policy          = policy_anything

## Generate Intermediate CA key

- Create intermediate CA key using 4096 bits and 3DES encryption

      [intermediate]# openssl genrsa -des3 -out private/intermediate.cakey.pem 4096

## Create immediate CA Certificate Signing Request (CSR)

- Create intermediate CA CSR with expiry value lesser than the root CA certificate

      [intermediate]# openssl req -new -sha256 -config openssl.cnf -key private/intermediate.cakey.pem -out csr/intermediate.csr.pem

**NOTE**: The Common Name must be different.

## Sign and generate immediate CA certificate

- Create immediate CA certificate with `v3_intermediate_ca` extension
  
      [intermediate]# openssl ca -config openssl.cnf -extensions v3_intermediate_ca -days 2650 -notext -batch -in csr/intermediate.csr.pem -out certs/intermediate.cacert.pem

## To change the format of the certificate to PEM format

- You might convert the format for Certificate Chain

      [intermediate] # openssl x509 -in certs/intermediate.cacert.pem -out certs/intermediate.cacert.pem -outform PEM

## Create more Intermediate CA certificates if needed

- Repeat actions from step 8 to step 11 to create more Intermediate CA certificates

- For the last intermediate certificate, can use the `v3_intermediate_ca_n` extension which defined as step 3.

## Sign and generate SGKLM server certificate

- Obtain the SGKLM Certificate Signing Request, can follow up <a href="https://www.ibm.com/docs/en/sgklm/4.1?topic=scenarios-scenario-request-third-party-certificate" target="_blank">SGKLM docs</a> to create a certificate request.

- Use the last immediate CA certificate and key to sign SGKLM CSR.

      [~/tls]# openssl x509 -req -days 730 -in 220215110925-sklm81-5.csr -CA certs/intermediate3.cacert.pem -CAkey intermediate/private/intermediate3.cakey.pem -CAcreateserial -out certs/sklm81-5.crt

- Import the signed certificate to SGKLM, can follow up <a href="https://www.ibm.com/docs/en/sgklm/4.1?topic=certificate-importing" target="_blank">SGKLM docs</a> to do that.

## References

<a href="https://w3.ibm.com/w3publisher/ibm-internal-certificate-authority" target="_blank">IBM Internal Certificate Authority</a> from w3Publisher   
<a href="https://www.golinuxcloud.com/openssl-create-certificate-chain-linux/" target="_blank">OpenSSL create certificate chain with Root & Intermediate CA</a>  
<a href="https://stackoverflow.com/questions/10175812/how-to-generate-a-self-signed-ssl-certificate-using-openssl" target="_blank">How to generate a self-signed SSL certificate using OpenSSL?</a>

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.