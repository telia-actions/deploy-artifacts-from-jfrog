
# Deploy artifacts to IIS using WebDeploy

This action can be used for downloading and extracting artifact from JFrog repository and installing to some IIS server.
It is using Powershell so is intended for Windows runners.

## Inputs

### local-storage-path:
  Optional path of local (usually self-hosted) runner's directory where artifacts might be stored after packaging them for faster access (instead of downloading from JFrog and extracting). Action continues with searching in JFrog if no matches found locally.
### jfrog-repo-name:
Name of JFrog repository to download from. Checking only local-storage-path if no value provided.
### jfrog-username:
  JFrog username to use for downloading artifact. Should have READ permissions.
### jfrog-password:
  JFrog user password.
### search-phrase:
  Substring of artifact name, usually short GIT SHA. Can be multiple search phrases delimited by triple pipe sign |||.
  In that case corresponding target parameters (runner-destination-path, server-iis-site-name or server-destination-path) should have the same number of values delimited by triple pipe sign |||.
  Either this or 'runner-source-path' is required.
### search-phrase-cleanup:
  Flag indicating if search-phrase input should be cleaned from spaces. Accepts "True" or "False" values. "True" by default
### archive-extension: **Required**
  Extension of artifact archive file. 
  Default: 'zip'
### runner-destination-path:
  Path to runner''s directory to download artifact to. Deployment part will be skipped if this parameter is provided. This is useful if you need to do some additional steps before deploying to the server.
### runner-source-path:
  Path to runner''s directory where artifact was previously downloaded. Local storage and JFrog are not checked if this parameter is provided. This is used as a deploy step if you need to do some addtional steps before deploying to the server (see parameter runner-destination-path).
### server-msdeploy-url: **Required**
  Url of server msdeploy, usually in format of https://some-server-name:8172/msdeploy.axd.
### server-msdeploy-username: **Required**
  Server username to use for deploying.
### server-msdeploy-password: **Required**
  Server user password to use for deploying.
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
        runs-on: [self-hosted, windows, example]
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


## Example with separate steps for download, modify and deploy

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
        runs-on: [self-hosted, windows, example]
        name: Download, modify and deploy artifact for some application
        steps:
          - uses: telia-actions/deploy-artifact-from-jfrog@v1
            with:
              local-storage-path: 'some-local-path-to-artifact'
              jfrog-repo-name: 'some-jfrog-repo-name'
              jfrog-username: ${{ vars.JFROG_USERNAME }}
              jfrog-password: ${{ secrets.JFROG_PASSWORD }}
              search-phrase: ${{ inputs.commit-hash }}
              runner-destination-path: '${{ hithub.workspace }}/artifact/'
              
          - name: Modify
            run: |
              echo "Some important modification to the downloaded artifact"
              
          - uses: telia-actions/deploy-artifact-from-jfrog@v1
            with:
              runner-source-path: '${{ hithub.workspace }}/artifact/'
              server-msdeploy-url: ${{ vars.SERVER_DEPLOY_URL }}
              server-msdeploy-username: ${{ vars.SERVER_DEPLOY_USERNAME }}
              server-msdeploy-password: ${{ secrets.SERVER_DEPLOY_PASSWORD }}
              server-iis-site-name: 'some-iis-application-name'