---
layout: post
title: Setting up Windows for Ansible
categories:
- ansible
---


[Ansible Official Documentation](http://docs.ansible.com/ansible/intro_windows.html#windows-system-prep) has a nice section about how to prepare windows machines. If you are not an winrm expert, you may encounter some issues that are hand-waivy in the documentation. It took me quite a lot of trial and error to figure out. 

Here are couple of steps to check: 

1. Make sure the windows account (Admin account) you are connecting to has a password.  
    
    Duh! but sometimes its not obvious, when you start with a passwordless test machine. Winrm will report error during configuration which took you a long time to figure out. 
    
2. Make sure all network connections are set to Home/Office Network. (namely, non-Public Network) 
    
    Sometimes its trickly to set unidentified networks, here are the steps. 
    1. Run gpedit.msc,  Computer Configuration > Windows Settings > Security Settings > Network List Manager Policies
    2. Set unindentified and public networks to private. See [official docs](https://technet.microsoft.com/en-us/library/jj966256.aspx)  
       
3. Sometimes you will get permission denied when running powershell script. Open powershell as admin and run the following to enable running scripts.

        set-executionpolicy remotesigned 
        
4. Now you can follow the [official guide](http://docs.ansible.com/ansible/intro_windows.html#windows-system-prep) to upgrade powershell and configure winrm. 
    
    There are 2 scripts: **upgrade_ps2_to_ps3** and **configureAnsibleforRemoting** to execute, plus a **hot patch** that must be applied. 


