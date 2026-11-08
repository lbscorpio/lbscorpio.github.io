---
title: Creating ROSA cluster with S3 bucket limit error
date: 2022-07-29 18:33:38
categories:
  - Troubleshoot
tags:
  - fusion
  - rosa
  - aws
---

In project Fusion SDS, our team often create a new ROSA cluster for testing, and will delete it once tasks complete, to avoid additional bill when the cluster is idle.

Today I hit a new creating error when creating a new ROSA cluster.

## Environment

- macOS Monterey 12.4
- rosa 1.1.12
- aws 2.5.6

## Issue

- Failed to create ROSA cluster, with error "Could not create S3 bucket as maximum buckets quota limit reached".

## Resolution

- By default, you can create up to 100 buckets in each of your AWS accounts. If you need additional buckets, you can increase your account bucket limit to a maximum of 1000 buckets by submitting a service limit increase.

- Or deleting the empty buckets if found, I chose this one in my case.

## Script to delete all empty S3 buckets

- <a href="https://s3.console.aws.amazon.com/s3/buckets" target="_blank">AWS S3 Console</a> can delete buckets, but fail to list all empty buckets directly.

- So I searched a quick way to list empty buckets and then delete them, the script as below:

      #!/bin/bash
      
      profile="default"
      
      emptybucketlist=()
      
      for bucketlist in  $(aws s3api list-buckets  --profile $profile  | jq --raw-output '.Buckets[].Name');
      do
        if [[ ! "$bucketlist" == *"shmr-logs" ]]; then
          listobjects=$(\
            aws s3api list-objects --bucket $bucketlist \
            --query 'Contents[*].Key' \
            --profile $profile)
      
          if [[ "$listobjects" == "null" ]]; then
                echo "$bucketlist is empty"
                emptybucketlist+=("$bucketlist")
          fi
        fi
      done
      
      echo -e "\n\n"
      
      echo "consider deleting these empty buckets"
      for emptybuckets in "${emptybucketlist[@]}";
      do
        aws s3api delete-bucket --profile $profile --bucket $emptybuckets
      done

- Before using above script, you laptop have to run `aws configure` firstly to ensure local have aws configuration.

- After running above script, AWS S3 Console might still see the deleted buckets list with region "Not found" after some time, but they will be gone completely after enough time(I am not sure the duration so far).

## References

- <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/BucketRestrictions.html" target="_blank">AWS S3 Bucket restrictions and limitations</a>  
- <a href="https://towardsthecloud.com/aws-cli-empty-s3-bucket" target="_blank">How to empty an S3 bucket using AWS CLI</a>  
- <a href="https://stackoverflow.com/questions/63301722/how-to-list-unused-aws-s3-buckets-and-empty-bucket-using-shell-script" target="_blank">How to list unused AWS S3 buckets and empty bucket using shell script?</a>

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.