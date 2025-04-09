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

Have a look in the config map described below

project : Backstage
config map : backstage-developer-hub-app-config

Around line 93 there are a number of catalog locations. Find a URL line with the target containing this path : summit-lab/backstage-workshops. Find that file in the GitLab repo. This file will list a number of target template files in the format shown below :

````
apiVersion: backstage.io/v1alpha1
kind: Location
metadata:
  name: showcase-templates
  description: A collection of Backstage templates for RH Summit
spec:
  type: url
  targets:
    - https://gitlab-gitl...
````

Find the GitHub URL for the template that you wish to add. If it is taken from this repo super-heroes / scaffolder-templates / super-heroes / template.yaml, then add the line below to the above location target list :

https://github.com/marrober/super-heroes/blob/main/scaffolder-templates/super-heroes/template.yaml

Add the above to the Locations yaml content in gitlab.

If you wait a few minutes then the template wil be visible in Developer Hub from the 'Create' menu. This then allows you to launch the template.

## Additional GitOps Instance

### ArgoCD configuration

Create a new ArgoCD instance in the openshift-gitops namespace. To do this follow the steps below :

- Select the openshift-gitops namespace in the OpenShift web UI
- Select Installed Operators from the left hand side admininstrator menu
- Select the Red Hat OpenShift GitOps operator
Select the ArgoCD sub menu
- Press the blue button marked "Create ArgoCD"
- Give the new instance the name 'main-gitops' and press 'Create'

Wait a minute or so for the new instance to spin up. 

- Select the new instance and then select the 'resources' sub menu.
- Find the 'main-gitops-server' route and open the URL to access the ArgoCD web UI.
- The username will be 'admin' and the password will be the same as the existing ArgoCD instances which is located in a secret called 'argocd-cluster'.

### DeveloperHub configuration

Update the DeveloperHub instance information for the new ArgoCD instance.

- In the gitlab instance open the repository : gitops/janus-idp-gitops
- Move to the directory : charts/backstage
- Open the file backstage-rhtap-values.yaml
- Edit the file in the GitLab repository and move down to line 315 to the argocd configuration information.
- Copy the four lines of the instance defintion and paste below the exiting entry.
- On the new instance block change the name to 'main-gitops' and change the URL to the appropriate URL for the new instance. Ensure the password remains as in the prior block.

Restart the DeveloperHub instance for the change to take effect by deleting the running DeveloperHub pod (for example backstage-developer-hub-nnnxxxhhh-fffgg)

Add a role binding to grant permission to the service account in the new ArgoCD instance to the backstage implementation :

````
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: main-gitops-backstage
  namespace: backstage
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: admin
subjects:
- kind: ServiceAccount
  name: main-gitops-argocd-application-controller
  namespace: openshift-gitops
````
