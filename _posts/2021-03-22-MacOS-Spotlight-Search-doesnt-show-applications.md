---
title:  MacOS Spotlight Search doesn't show applications
date: 2021-03-22 13:20:20
categories:
  - Troubleshoot
tags:
  - macos
  - sip
  - spotlight
---

My laptop can't list applications(e.g. calculator or textedit) on the Spotlight Search, it happened after some upgrade.

## Environment

- MacOS Big Sur 11.x

## Issue

- After launching Spotlight Search (Shortcut: `Command` + `Space` bar), enter an application keyword `calc`, it only shows the definition list, can't see a such application what I want as normal.

## Resolution

- Disable System Integrity Protection (SIP) firstly
  
  - Restart your computer in <a href="https://support.apple.com/en-us/HT201314" target="_blank">Recovery mode</a>
    
  - Launch Terminal from the Utilities menu
    
  - Run the command `csrutil disable`
    
  - Restart your computer
    
- Rebuild the index
 
  - Turn off Spotlight by issuing command `sudo mdutil -a -i off`
    
  - Unload the metadata file that controls Spotlight’s index
        
    ```zsh
    sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.metadata.mds.plist
    ```
    
  - Re-load the metadata file
    
    ```zsh
    sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.metadata.mds.plist
    ```
   
  - Turn Spotlight back on by issuing command `sudo mdutil -a -i on`

- Enable System Integrity Protection (SIP) again

  - Restart your computer in <a href="https://support.apple.com/en-us/HT201314" target="_blank">Recovery mode</a>
    
  - Launch Terminal from the Utilities menu
    
  - Run the command `csrutil enable`
    
  - Restart your computer

## References

<a href="https://apple.stackexchange.com/questions/368054/spotlight-doesnt-show-applications" target="_blank">Spotlight doesn't show applications</a>  
<a href="https://support.apple.com/en-us/HT201236" target="_blank">Mac keyboard shortcuts</a>  
<a href="https://developer.apple.com/documentation/security/disabling_and_enabling_system_integrity_protection" target="_blank">Disabling and Enabling System Integrity Protection</a>  

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.