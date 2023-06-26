# DEPLOY GITLAB USING ANSIBLE

### **Before proceeding via automation**

1. Ensure to install pre-requisites to run the gitlab automation. The following pip package modules need to install.
    - kubernetes
    - kubernetes-validate
    - cryptography

2. The following ansible galaxy collections need to install,
    - community.crypto
    - kubernetes.core.k8s

3. Also, set the kube-config environment on to the deployable machine.
    ```
    - mkdir -p ~/.kube
    - cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
    - export KUBECONFIG=~/.kube/config
    ```

    ```
    Note: Export `KUBECONFIG` to the environment would be add to setip script
    ```

### **Ansible Vault Variable Environment Setup**

1. Export Ansible vault password file, so that it will load while running the ansible playbook.

    ```export ANSIBLE_VAULT_PASSWORD_FILE=/opt/hpe/Bootstrap/vault_pass.json```

### **Installation of the GITLAB**

1. Edit inventory file and do changes as per the hosts

2. Run the ansible playbook to install gitlab with the skip tags "upgrade,undeploy"
    - ansible-playbook -i hosts gitlab_playbook.yml --skip-tags "upgrade,undeploy" --vault-password-file /opt/hpe/Bootstrap/vault_pass.json
    
    ```
    Note:
    - If we setup the **ANSIBLE_VAULT_PASSWORD_FILE** environment then it can run with below playbook command
    - ansible-playbook -i hosts gitlab_playbook.yml --skip-tags "upgrade,undeploy"     
    ```
### **Extra variables to run the GITLAB**

Optional extra vars:
 - config_storage: 3Gi 
 - data_storage: 3Gi  
 - logs_storage: 3Gi
 - ssl_storage: 3Gi

### **Upgrade the GITLAB**

1. Run the ansible playbook with the tag "upgrade" to upgrade the gitlab
    - ansible-playbook -i hosts gitlab_playbook.yml --tags "upgrade" --vault-password-file /opt/hpe/Bootstrap/vault_pass.json
    
    ```
    Note:
    - If we setup the **ANSIBLE_VAULT_PASSWORD_FILE** environment then it can run with below playbook command
    - anisble-playbook -i hosts gitlab_playbook.yml --tags "upgrade"
    ```

Required Extra vars:

 - gitlab_upgrade_version: "15.5.1-ce.0"  # image name along with version

### **Uninstall the GITLAB**

1. Run the ansible playbook to unistall the gitlab with the following tags "undeploy"
    - ansible-playbook -i hosts gitlab_playbook.yml --tags "undeploy" --vault-password-file /opt/hpe/Bootstrap/vault_pass.json
    
    ```
    Note:
    - If we setup the **ANSIBLE_VAULT_PASSWORD_FILE** environment then it can run with below playbook command
    - anisble-playbook -i hosts gitlab_playbook.yml --tags "undeploy"
    ```

