# Overview

Automatically configure new OpenShift projects, and select pods in a project, with common assets required to interact with internal resources (e.g. services using self-signed certificates). These steps apply to OpenShift v3, but should be compatible with OpenShift 4 as well.

# Configuration

  1. Apply the appropriate ClusterRole and CRD to support PodPresets
  ```
  oc apply -f https://raw.githubusercontent.com/redhat-cop/podpreset-webhook/master/deploy/crds/redhatcop_v1alpha1_podpreset_crd.yaml
  oc apply -f https://raw.githubusercontent.com/redhat-cop/podpreset-webhook/master/deploy/clusterrole.yaml
  ```
  2. Configure a project request template to stage objects/resources/definitions required by PodPreset CustomResources
  ```
  oc apply -f project-template.yaml -n default
  
  # Create/modify your own, but make sure to include appropriate role, rolebinding, secret, deployment, etc. details from the sample project-template.yaml
  # Example:
  # oc adm create-bootstrap-project-template -o yaml > project-template-base.yaml
  ```
  3. Associate the created ClusterRole with future serviceAccount and create a new project using the project request template
  ```bash
  export PROJECT_NAME="sample"
  oc adm policy add-cluster-role-to-user podpreset-webhook system:serviceaccount:${PROJECT_NAME}:podpreset-webhook
  oc new-project ${PROJECT_NAME}
  ```
  4. (Optional) Create sample workloads
  ```
  oc apply -f deployment-sample.yaml -n ${PROJECT_NAME}
  ### AND/OR 
  oc new-app ruby~https://github.com/sclorg/ruby-ex.git -n -n ${PROJECT_NAME}
  ```
  4. Label a supported resource to test PodPresets - optionally, add the labels in your source deployment/deploymentConfig manifests
  ```
  oc patch deployment/ocp-ftw -p '{"spec":{"template":{"metadata":{"labels":{"workload":"intra"}}}}}' -n ${PROJECT_NAME}
  ```

# Further Configuration

  - Consider using [namespace-configuration-operator](https://github.com/redhat-cop/namespace-configuration-operator) over project request templates, particularly for OpenShift 4
  - Consider using [Ansible and the k8s module](https://docs.ansible.com/ansible/latest/modules/k8s_module.html) to template object creation on OpenShift
    > This is especially valuable if integrating project bootstrapping into a self-service catalog (e.g. ServiceNow, CloudForms, etc.)
