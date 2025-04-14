# Superheroes Web Template

This repository contains the Backstage Template used to create the Kubernetes resources needed to build/deploy a simple quarkus application.

## Developer Hub Setup

### Template edits

Since the template is running in a new cluster it is necessary to update the URLs. 

Update : 
- quay url
- gitlab urls (two)
- cluster url (in the two 'fetch:template' steps)

### Add Template to Developer Hub

To add the template in this repository to a Developer Hub instance it is necessary to add a link to a file in the GitLab repo. 

Update the file in gitlab : gitops / janus-idp-gitops / charts / backstage / backstage-values.yaml

Add the following catalog rule, ensuring that the target points to your repo if you have cloned a copy from marrober repo.

````
      - rules:
        - allow:
          - Template
        target: https://github.com/marrober/super-heroes/blob/main/scaffolder-templates/templates-library.yaml
        type: url
````

There are multiple instances of ArgoCD on the cluster. In the openshift-gitops namespace select the ArgoCD server with the URL that begins wuith argocd-server-openshift-gitops.

Force a hard refresh and a sync of the backstage ArgoCD application.

If you wait a few minutes then the template wil be visible in Developer Hub from the 'Create' menu. This then allows you to launch the template.
