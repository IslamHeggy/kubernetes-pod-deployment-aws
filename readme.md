Pod Kubernetes Deployments
==============================

This repository contains ansible code to automate the pod deployments on kubernetes cluster with all the needed security groups, load balancers, domains and waf rules.

## Table of Contents
* **[Prerequisites](#pre)**
* **[How does it work?](#how)**
* **[Create New Deployment](#deploy)**
* **[Edit Already Existing Deployment](#edit)**
* **[Troubleshooting](#trouble)**

## Prerequisites
Before running the Ansible code you should install the following packages with the specified versions or later on the Ansible machine.

* Ansible 2.7.0+
* aws cli 1.16.28+
* Python 2.7.15+
* boto3 1.12.24+
* botocore 1.12.24+
* boto 2.49.0+
* openshift 0.7.2+
* yq 2.1.2+
* jq 1.5-1+

**NOTE:** The code is tested with the exact versions mentioned above on Linux Mint 19.

## How does it work?

This Ansible code does the following functionalities

* Create a new user and database for the deployment.
* Deploy pods.
* Create internet facing ALB for the deployment.
* Create internal ALB for the deployment
* Add subdomain for the deployment.
* Attach existing WAF rule.
* Create security groups.

It has 2 scenarios to go through

* One to create a new deployment (refer to Create New Deployment part for more info).
* The other is to edit the current deployment (refer to Edit Already Existing Deployment part for more info).   


## Create New Deployment

To start a fresh deployment with Ansible you should first adjust some points according to your deployment.

1. Copy the example-client.yml variables file in the same directory with your client or service name. ex: XCLIENTX.yml

2. After that edit the new file with the client variables such as client_namespace, client_pods and service reserved ports vars.

3. Stop editing the file when you find **STOP EDITING HERE** line.

4. Edit the all.yml file and place your key for connecting to the node that have network access to the rds instances.

```
ansible_ssh_private_key_file: ~/.ssh/test.pub

```
5. After that you can run the following command in the ansible directory while assigning the set_client and client_create_internal_alb per your need.

**NOTE:** set_client variable takes the variables file you edited before, so in our case it's XCLIENTX.yml.
		  The client_create_internal_alb can take yes-http, yes-https and no for creating an http or https internal load balancer respectively.
		  If any of these variables are not set it will cause playbook **failure**.

```
ansible-playbook playbooks/deploy-pod.yml  -e "set_client= XCLIENTX.yml client_create_internal_alb=yes-http" -i Inventory/myenvironment/

```

## Edit Already Existing Deployment

So in case of you want to edit your current deployment by changing the replicas or even changing images tags for adding new features or any other edit, you have to take the following points into consideration.

* Edit the template file and the variables file of your client with your new changes.
* Delete the previously created files in the files directory so Ansible can generate the new files with the new deployment parameters.
* Now you are ready to run the the playbook to adjust your deployment, refer to Create New Deployment section point 4 and continue the deployment steps.


## Troubleshooting
The deployment can fail in several cases so here are some steps to debug your deployment in case if failure so here are some specific cases.

* The deployment fails when the node ports already exist.
  * This can happen when the script deploys the service part with the node range and fails or be stopped before the running the deployment template and then it will always go through the new deployment with the same assigned ports in a previous uncompleted deployment, so you can delete this service manually or assign a new port range.

  * This can happen when you use already used ports in a current deployemnt, so you can assign new port ranges in the variables file.

```
causes\":[{\"reason\":\"FieldValueInvalid\",\"message\":\"Invalid value: 30001: provided port is already allocated\",\"field\":\"spec.ports[0].nodePort\"}]},\"code\":422}\n",
    "reason": "Unprocessable Entity",
    "status": 422
```


* The deployment failed in creating target group part.
  * This is most probably because there is an existing target group with the same name, so you can delete the alb and the target group and try redeploying again.

General Troubleshooting points.

* If you faced module failure with a command not found or can't execute command so it might be a missing python package so make sure you install all the needed packages mentioned in the prerequisites part.

* If you have any unusual behavior and you need to debug any part you can comment all the included functions in the main.yml put a debug for your needed variables and start debugging.
  * Take care not to comment dependent parts so it won't fail.

* In case of editing the deployment if nothing changes after changing the template this is most probably the files are not changed so delete the old files so the the new files are created in the files directory.
