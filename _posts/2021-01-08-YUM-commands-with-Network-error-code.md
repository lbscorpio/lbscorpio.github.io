---
title:  YUM commands with network error code 400
date: 2021-01-08 16:42:20
categories:
  - Troubleshoot
tags:
  - linux
  - yum
  - rhsm
---

When I tried to run some yum commands on a testing node, can see the warning "Network error code: 400" every time.

## Environment

- Red Hat Enterprise Linux 7.x

## Issue

- YUM commands reported "Network error code: 400" every time.

      # yum repolist
      Loaded plugins: product-id, search-disabled-repos, subscription-manager
      Network error code: 400               <<<<<

## Resolution

- Cleaning the subscription-manager data, clears old certificate files and corrupt data.

      # subscription-manager clean
      All local data removed
      
      # subscription-manager status
      +-------------------------------------------+
         System Status Details
      +-------------------------------------------+
      Overall Status: Unknown

## Diagnostic Steps

- Traceback found in /var/log/rhsm/rhsm.log file:

      2021-01-08 03:33:15,841 [INFO] yum:18651:MainThread @connection.py:830 - Connection built: host=subscription.rhsm.redhat.com port=443 handler=/subscription auth=identity_cert ca_dir=/etc/rhsm/ca/ verify=False
      2021-01-08 03:33:15,845 [INFO] yum:18651:MainThread @connection.py:830 - Connection built: host=subscription.rhsm.redhat.com port=443 handler=/subscription auth=identity_cert ca_dir=/etc/rhsm/ca/ verify=False
      2021-01-08 03:33:16,386 [ERROR] yum:18651:MainThread @connection.py:631 - Response: 400
      2021-01-08 03:33:16,386 [ERROR] yum:18651:MainThread @connection.py:632 - JSON parsing error: No JSON object could be decoded
      
## Reference

<a href="https://access.redhat.com/solutions/5187461" target="_blank">Subscription-manager command fails with "NetworkException: Network error code: 400"</a>   

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.