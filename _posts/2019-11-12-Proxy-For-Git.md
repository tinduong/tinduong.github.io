---
title: Proxy for Git
tags: [Git],[Proxy]
style: fill
color: warning
description: Setting up your proxy for vs code to work with git repos.
---

# Proxy for git

## Azure DevOps

Have your own azure devops account and trying to access it within a company network? you can install [git credential](https://github.com/Microsoft/Git-Credential-Manager-for-Windows) (GCM) and let the magic happens. When you try to clone it, there will be a login prompt to enter username and password. However, if for some reasons, that does not work. Here is how to do it with git command

* Go to yourorg.visualstudio.com/_usersSettingstokens
* Generate your PAT (Personal Access Token)
* Clone your Repository with this command:

```git
git clone https://anything:yourpersonalaccesstokengoeshere@dev.azure.com/YourOrganization/YourProjectName/_git/RepoName
```

Also check out [Microsoft official documentation](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=preview-page)

## GitHub

Use these commands to set the config for your proxy

### Command line

```git
git config --global http.sslVerify false
git config --global https.sslVerify false
git config --global http.proxy http://username:password@yourproxyserver:portnumber
git config --global https.proxy http://username:password@yourproxyserver:portnumber
```

### VS Code settings

* Ctrl + Shift + P
* Enter 'Open Settings JSON'
* Add these lines

```json
// Place your settings in this file to overwrite the default settings
{
    "http.proxy": "http://username:password@proxyserver:portnumber",
    "http.proxyStrictSSL": false,
}
```
