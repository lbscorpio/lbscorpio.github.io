---
title:  Steps to contribute on Open Source projects
date: 2021-02-23 13:28:51
categories:
  - Procedure
tags:
  - github
  - opensource
  - git
---

Many years ago, I would like be a coding guy but not taking actions for any Open Source projects, because of my limited coding experience. 

## Prerequisites

- Have taken the required <a href="https://ibm.biz/BdzGnB" target="_blank">annual open source training</a>  
- Fixing a typo or cleaning up some grammar in a comment or documentation does not require a formal request. Others should apply contribution requests in WICKED, refer to <a href="https://w3.ibm.com/developer/opensource/contribute/contributions/" target="_blank">here</a> for more information.
- The existing github accounts, not only public GitHub account, but also GitHub Enterprise account.
- Have linked your public GitHub ID to your IBM ID using <a href="http://ibm.biz/gh-user-mapper" target="_blank">GitHub User Mapper</a> webpage.
- Setup authentication between your workstation and GitHub, can refer <a href="https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh" target="_blank">Connecting to GitHub with SSH</a> for more information.

## Procedures

### Fork and clone your interesting project

- Search your interesting repository from GitHub UI, then clicking on **Fork** button in the upper right-hand corner.

- Open a terminal and change into the directory where you want to clone the forked repository.

- Clone the forked repository onto your workstation with the following command(taking openshift-docs repository as example), replacing <user_name> with your actual GitHub ID.

      # git clone git@github.com:<user_name>/openshift-docs.git
      
- Change into the directory for the local repository you just cloned.

      # cd openshift-docs
      
- Add an upstream pointer back to the OpenShift’s remote repository, in this case _openshift-docs_.

      # git remote add upstream git@github.com:openshift/openshift-docs.git

### Ensure your local repository is in sync with the remote

- From your local repository, make sure you have the `master` branch checked out.

      # git checkout master

  If you want to change the contents based on another remote branch, create a local branch which track a remote branch.

      # git checkout --track origin/<branch_name>

- Fetch the current state of the OpenShift documentation repository

      # git fetch upstream

- Incorporate the commits from the remote repository, in this case `openshift/openshift-docs`, into your local repository.

      # git rebase upstream/master

- Push the latest updates to your forked repository so that it is also in sync with the remote

      # git push origin master

### Add content or update existing content on local branch

- Create local working branch where you will make all your updates, or create any new content.

      # git checkout -b <working_branch>

- Create content or update existing content as required, ensure following up documentation guidelines if have.

- Add all of your changes to a pending commit

      # git add .

- Commit your changes

      # git commit -am "Detailed comments about what changes were made; for example, fixed typo"

- Rebase updates from `master` into your working branch

      # git rebase upstream/master

  > If you find any conflicts you must fix those, and repeat previous two steps.

- Push all changes to your GitHub account

      # git push origin <working_branch>

### Submit PR to merge your work

- Go to your forked GitHub repository on the GitHub website, and you should see your local branch that includes all of your work.

- Click on **Pull Request** to submit the PR against the `master` branch or another remote branch of the forked repository.

### Confirm your changes have been merged

- Just repeat the same steps of "Ensure your local repository is in sync with the remote".

### Add changes to an existing PR, if required

In some cases you might have to make changes to a PR that you have already submitted.

- Commit whatever updates you have made to the working branch by creating a new commit

      # git commit -am "Detailed message as noted earlier"

- Rebase your PR and squash multiple commits into one commit. 

      # git rebase -i HEAD~n
 
  > Where `n` is the number of commits to rebase.

- Rebase new updates from `master` into your working branch in case already have

      # git rebase upstream/master

  If your commits based on another remote branch, issuing the following command:

      # git rebase upstream/<branch_name>

- After you have rebased and squashed, push the latest updates to the local working branch to your GitHub account.

      # git push origin <working_branch> --force

### Delete the local working branch

When you have confirmed that all of your changes have been accepted and merged, and you have pulled the latest changes on master and pushed them to your GitHub account, you can delete the local working branch. 

- Delete the local working branch from your workstation
 
      # git branch -D <working_branch>

- Delete the working branch from your GitHub account

      # git push origin :<working_branch>

## References

<a href="https://w3.ibm.com/developer/opensource/" target="_blank">Open Source@IBM</a> from W3 Developer  
<a href="https://w3.ibm.com/w3publisher/ossc-process" target="_blank">Open Source Steering Committee</a> from w3Publisher  
<a href="https://w3.ibm.com/w3publisher/contributing-to-open-source" target="_blank">Contributing to Open Source</a> from w3Publisher  

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.