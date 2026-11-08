---
title:  Failed to login GKLM GUI after few months
date: 2022-08-25 20:15:10
categories:
  - Troubleshoot
tags:
  - fusion
  - gklm
  - sklm
---

Our project need to test IBM Spectrum Scale filesystem encryption function, so I did setup a GKLM environment as the key server.

But it will be unavailable after few months, the root cause is unknown yet.

## Environment

- RHEL 7.9
- GKLM 4.1.1

## Issue

- Failed to login the GKLM GUI with errors "Database connection failed on data source. Check if database server needs to be restarted or use recovery mechanism to update database password."

## Resolution

- Update Db2 password on GKLM server after login as root

  ```
  # su klmdb411
  $ passwd
  $ exit
  ```

- Update the new Db2 password in the configuration file manually

  - Encode the new password firstly
     
    ```
    # /opt/IBM/WebSphere/Liberty/bin/securityUtility encode --encoding=custom <new password to encrypt>
    ```    

  - Update the encrypted password for user `klmdb411` in the file datasource.xml
    
    ```
    # vi /opt/IBM/WebSphere/Liberty/usr/servers/defaultServer/datasource.xml
    ```
    
- Restart the Liberty server, then wait few minutes.

    ```
    # /opt/IBM/WebSphere/Liberty/bin/restartLibertyServer.sh /opt/IBM/WebSphere/Liberty
    ```

- Before login the GKLM GUI, can double check if the Db2 available or not

  - Login to the GKLM server using user `klmdb411`
    
  - Run below command to check if can connect the database
    
    ```
    $ db2 connect to klmdb411
    
       Database Connection Information

     Database server        = DB2/LINUXX8664 11.5.6.0
     SQL authorization ID   = KLMDB411
     Local database alias   = KLMDB411
    ```
    
  - If fail to connect the Db2, can try to restart it using below commands.
    
    ```
    $ /home/klmdb411/sqllib/db2stop.sh
    $ /home/klmdb411/sqllib/db2start.sh
    ```

## References

<a href="https://sklm.slack.com/archives/C8PLBURPF" target="_blank">SKLM Slack channel</a>  
<a href="https://www.ibm.com/docs/en/sgklm/4.1.1?topic=password-updating-db2-linux-aix-system" target="_blank">Updating Db2 password on a Linux or AIX system</a>  
<a href="https://www.ibm.com/support/pages/db2start-fails-sql10007n-message-1390-could-not-be-retrieved-reason-code-3" target="_blank">DB2START Fails with SQL10007N Message "-1390" could not be retrieved. Reason code: "3"</a>  

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.