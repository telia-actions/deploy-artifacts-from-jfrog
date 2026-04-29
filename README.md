
# Deploy artifacts to IIS using WebDeploy

This action can be used for downloading and extracting artifact from JFrog repository and installing to some IIS server.
It is using Powershell so is intended for Windows runners.

## Inputs

### jfrog-repo-name:
Name of JFrog repository to download from. Checking only local-storage-path if no value provided.
### jfrog-username:
  JFrog username to use for downloading artifact. Should have READ permissions.
### jfrog-password:
  JFrog user password.
### search-phrase:
  Substring of artifact name, usually short GIT SHA. **Required**
### search-phrase-cleanup:
  Flag indicating if search-phrase input should be cleaned from spaces. Accepts "True" or "False" values. "True" by default
### archive-extension:
  Extension of artifact archive file. **Required**
  Default: 'zip'
### server-msdeploy-url:
  Url of server msdeploy, usually in format of https://some-server-name:8172/msdeploy.axd. **Required**
### server-msdeploy-username:
  Server username to use for deploying. **Required**
### server-msdeploy-password:
  Server user password to use for deploying. **Required**
### server-iis-site-name:
  Name of the IIS site on the server. Either this or server-iis-site-name is required.
### server-destination-path:
  Path on server to install application to. Either this or server-iis-site-name is required.
### presync-command-file-path:
  Path to cmd file to run before installation.
### postsync-command-file-path:
  Path to cmd file to run after installation.
### application-path:
  Optional sub-path of application in JFrog artifact (in case archive contains more than one application or application source is in some sub-folder).
### msdeploy-skip-parameter:
  Optional MsDeploy -skip parameter value. For example, `objectName=dirPath,absolutePath=Logs`, or `objectName=filePath,absolutePath=site.config`.
  For multiple values pass semicolon separated values, for example, `objectName=appPool;objectName=application`.
### local-storage-path:
  Optional path of local (usually self-hosted) runner's directory where artifacts might be stored after packaging them for faster access (instead of downloading from JFrog and extracting). Action continues with searching in JFrog if no matches found locally.


## Example

```
    name: Artifact deploy for some application
    
    on:
    workflow_dispatch:
        inputs:
          commit-hash:
            description: 'Short commit version (7 digits) which will be deployed. If nothing is entered the latest will be deployed.'
            required: true
    
    jobs:
      deploy artifact:
        runs-on: [ec2, windows, medium]
        name: Download and deploy artifact for some application
        steps:
          - uses: telia-actions/deploy-artifact-from-jfrog@v1
            with:
              jfrog-repo-name: 'some-jfrog-repo-name'
              jfrog-username: ${{ vars.JFROG_USERNAME }}
              jfrog-password: ${{ secrets.JFROG_PASSWORD }}
              search-phrase: ${{ inputs.commit-hash }}
              server-msdeploy-url: ${{ vars.SERVER_DEPLOY_URL }}
              server-msdeploy-username: ${{ vars.SERVER_DEPLOY_USERNAME }}
              server-msdeploy-password: ${{ secrets.SERVER_DEPLOY_PASSWORD }}
              server-iis-site-name: 'some-iis-application-name'
```
