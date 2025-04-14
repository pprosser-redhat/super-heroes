# Superheroes Web Template

This repository contains the Backstage Template used to create the Kubernetes resources needed to build/deploy a simple quarkus application.

## Quay repository setup

Create a new quay repository called  quayadminsuperheroes-rest-heroes and make it public.

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

## Stackrox job setup 

To be added back into job-setup-stackrox.yaml when ACS config is required

apiVersion: batch/v1
kind: Job
metadata:
  name: {{ .Values.app.name }}-stackrox-setup-job
  annotations:
    argocd.argoproj.io/sync-wave: "3"
spec:
  backoffLimit: 100
  template:
    spec:
      containers:
      - name: setup-stackrox
        command:
          - /bin/bash
          - '-c'
          - |
            set -x
            cd /scripts
            ansible-playbook -i localhost playbook.yaml \
            -e central_stackrox_url=$CENTRAL_STACKROX_URL \
            -e central_stackrox_password=$COMMON_PASSWORD \
            -e component_id={{ .Values.app.name }} \
            -e apps_domain={{ .Values.app.cluster }}
        image: quay.io/agnosticd/ee-multicloud:latest
        volumeMounts:
          - mountPath: /scripts
            name: {{ .Values.app.name }}-script-vol
        env:
          - name: COMMON_PASSWORD
            valueFrom:
              secretKeyRef:
                key: password
                name: common-password-secret
          - name: CENTRAL_STACKROX_URL
            value: https://central-stackrox{{ .Values.app.cluster }}

      restartPolicy: Never
      volumes:
      - name: {{ .Values.app.name }}-script-vol
        configMap:
          name: {{ .Values.app.name }}-stackrox-setup-script
      serviceAccount: job-runner
