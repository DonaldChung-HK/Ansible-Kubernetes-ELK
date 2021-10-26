# Ansible-Kubernetes-ELK
In a production environment, Kubernetes is a highly dynamic and distributed environment across multiple machines and containers. This poses a unique challenge for logging and gaining visibility into the system.
ELK stack is a popular choice for open-sourced log management. Deploying an ELK stack on Kubernetes is very useful for monitoring and log analysis purposes. However, it can be a complex task, and this playbook is designed to streamline this process for rapid deployment of ELK stacks in Kubernetes cluster.

## Features
- Deploy entire ELK stacks with a single playbook
- version management to ensure version consistancy across the stack
- Options to install with logstash or send logs directly form filebeat to elastic search
- Support for persistent volumes claim to prevent accidental data loss due to pod termination

## Requirements
- Ansible
- Python 3
	- Packages:
		- ansible
		- setuptools
		- setuptools-rust
		- openshift
- kubectl
- Helm 3
- Persistent Volumes and Load Balancer provider on K8s Cluster

## Preperation
- Install python 3 and required packages 
	- `pip3 install ansible setuptools setuptools-rust openshift`
- Install ansible, kubectl and helm 3
	- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
	- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
	- [helm](https://helm.sh/docs/intro/install/)
- Setup `$KUBECONFIG` for kubernetes cluster access
- Install Ansible requirements
	- `ansible-galaxy collection install -r requirements.yml`
- Place a yaml file storage provider of your Kubernetes cluster in `/roles/prep/files/`

## Parameters
### Basics parameters
These can be configured in `playbook.yaml`.

| **Variables** | **Description** | **Default Value**
| ------------ | ------------ | ------------ |
| `deployment_namespace`  |  Namespace to deploy the ELK stacks. This will be created automatically dueing deployment if it doesn't exist already. | `logs` |
| `deploy_with_logstash`  | whether to install with logstash (true) or send logs directly form filebeat to elastic search (false). Logstash is usually deployed for log processing features that are not avaliable with filebeat. | `true`  |
| `storage_class_name` | Name of storage class for PV claims | `"local-path"` | 
| `storage_class_file` | Name of StorageClass yaml file placed in `/roles/prep/files/`. | `"local-path-storage.yaml"` |
| `image_version` | Image tag of the image to pull. The same version will be installed across all component for better compativbility. | `"7.15.1"` |
| `elasticsearch_replica` | Number of elastic search replicas to deploy. | `3` |
| `min_master_node` | Minimum number of elastic search master node. | `2` |
| `elastic_storage` | Size of PV claims for elastic search pods | `30Gi` |
| `logstash_replica` | Number of logstash replicas to deploy. | `1` |
| `logstash_storage` | Size of PV claims for logstash pods | `5Gi` |
### Config Files
Advanced configurations can be modiflied in the YAML files in `/role/{component_name}/files/`
#### Elasticsearch
Filebeat options can be modiflied in `/role/elastic_search/files/elastic_search_values.yml`

| Variables  |  Description |
| ------------ | ------------ |
| `esJavaOpts`  |  Java options for Elasticsearch such as heap size |
| `resources`  |  resource for each Elasticsearch |
|  `esConfig: elasticsearch.yml` |  Config file for Elasticsearch xpack security is explicitly disabled to disable warnings |

#### Filebeat
Filebeat options can be modiflied in `/role/filebeat/files/filebeat_values.yml` or `/role/filebeat/files/filebeat_values_with_logstash.yml` depending on weather `deploy_with_logstash` is set to true.

| Variables  |  Description |
| ------------ | ------------ |
| `resources`  |  resource for each Filebeat |
|  `daemonset: filebeatConfig: filebeat.yml:` `deployment : filebeatConfig: filebeat.yml:`|  Config file for filebeat to modifly log input and processors |

#### Logstash
logstash options can be modiflied in `/role/logstash/files/logstash_values.yml` or `/role/filebeat/files/filebeat_values_with_logstash.yml`

| Variables  |  Description |
| ------------ | ------------ |
| `logstashPipeline: logstash.conf:`|   Config file for logstash pipeline such as filters for incomming logs |

## Post-installations
To connect to Kibana dashboard, check the external IP using `kubectl get services -A` and open `http://<externam-ip-of-kibana>:5601`
