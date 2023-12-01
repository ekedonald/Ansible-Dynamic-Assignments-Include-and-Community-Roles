# Ansible Dynamic Assignments (Include) and Community Roles
The [Ansible Automation](https://github.com/ekedonald/Ansible-Automation-Project) and [Ansible Refactoring Assignments and Imports](https://github.com/ekedonald/Ansible-Refactoring-Assignments-and-Imports) projects have already explored how to perform configurations using `playbooks`, `roles` and `imports`. In this project, I will introduce [Dynamic Assignments](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse.html#includes-dynamic-re-use) by using the `include` module. From the Ansible Refactoring Assignment project, it is evident that **Static Assignments** use the `import` module. However, the module that enables **Dynamic Assignments** is `include`.

When the `import` module is used, all statements are pre-processed at the time playbooks are parsed (i.e. when you execute `site.yml` playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements). This also means that during actual execution, if any statement changes, such statements will not be considered. Hence, it is **Static**.

On the other hand, when the `include` module is used, all the statements are processed only during the execution of the playbook (i.e. after the statements are parsed, any changes to the statements encountered during execution will be used).

However, it is recommended to use **Static Assignments** for playbooks because it is more reliable while **Dynamic Assignments** are difficult to debug due to their dynamic nature. Dynamic Assignments can be used for environment-specific variables.

## How To Implement Ansible Dynamic Assignments (Include) and Community Roles
The following steps are taken to Implement Ansible Dynamic Assignments (Include) and Community Roles:

### Step 1: Introduce Dynamic Assignment into your File Structure

* Create and switch into the `dynamic-assignments` branch.

```sh
git checkout -b dynamic-assignments
```

* Create a `dynamic-assignments` directory and create an `env-vars.yml` file inside the directory.

```sh
mkdir dynamic-assignments && cd dynamic-assignments && touch env-vars.yml
```

* Update the `env-vars.yml` file with the following codebase:

```sh
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yml
            - stage.yml
            - prod.yml
            - uat.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always
```

There are 3 things to notice in the codebase above:

1. The `include_vars` module was used instead of `include` because Ansible developers decided to separate different features of the module. From Ansible version 2.8, the `include` module was depreciated and variants of `include_*` must be used (i.e. [include_role](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_role_module.html#include-role-module), [include_tasks](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_tasks_module.html#include-tasks-module) and [include_vars](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_vars_module.html#include-vars-module)).

2. The following [special variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html) were used: `{{ playbook_dir }}` and `{{ inventory_file }}`.
The `{{ playbook_dir }}` helps Ansible determine the location of the running playbook and navigation to other paths on the filesystem while the `{{ inventory_file }}` will dynamically resolve the name of the inventory file being used then append `.yml` so that it picks up the required file within the `env-vars` directory.

3. The variables are included using the `with_first_found` loop which is used to loop through the list of files and the first one is used. This is good so that we can always set default values in case an environment-specific `env` file does not exist.

* Go a step back in the directory, create an `env-vars` directory and create the following files: `dev.yml`, `stage.yml`, `uat.yml` and `prod.yml` inside the directory.

* The layout of the `ansible-config-mgt` directory should like this:


### Step 2: Update `site.yml` with Dynamic Assignments

* Update the `site.yml` playbook configuration file to import files from the `dynamic-assignments` directory.

```sh
---
- hosts: all
  become: true
  name: Import Dynamic Variables
- import_playbook: ../dynamic-assignments/env-vars.yml
```

### Step 3: Merge the changes from the dynamic-assignments branch into the main branch

* Run the following command to view the untracked files (i.e. the file and directory you just created):

```sh
git status
```

* Add the untracked files and commit the changes.

```sh
git add playbooks/site.yml dynamic-assignments env-vars && git commit -m "updates"
```

* Push all changes from the `dynamic-assignments` branch to the `main` branch.

```sh
git push --set-upstream origin dynamic-assignments
```

* Go to your `ansible-config-mgt` repository on GitHub and click on the `Compare & pull request` button.

* Click on the `Create pull request` button.

* Click on the `Merge pull request` button.

* Click on the `Confirm merge` button.

* Go to the `ansible-config-mgt` directory on your local machine and run the following command to switch to the `main` branch and pull the changes:

```sh
git checkout main && git pull
```

### Step 4: SSH into the Jenkins-Ansible server, pull files from the `ansible-config-mgt` repository and create a `roles-feature` branch

* SSH into the `Jenkins-Ansible` server.

```sh
ssh ubuntu@<public_ipv4_address_of_jenkins_server
```

* Check if git is installed on the server.

```sh
git --version
```

* Create and go into the `ansible-config-mgt` directory.

```sh
mkdir ansible-config-mgt && cd ansible-config-mgt
```

* Run the following commands to initialize a repository, pull files from your `ansible-config-mgt` repository on GitHub, add a new remote repository reference, create and switch to the `roles-feature` branch.

```sh
git init
git pull https://github.com/<your-name>/ansible-config-mgt.git
git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
git checkout -b roles-feature
```

### Step 5: Using Ansible-Galaxy, Download Community Roles for Apache, Nginx and MySQL into the roles directory

* Go into the `roles` directory.

```sh
cd roles
```

* Create a MySQL role with Ansible-Galaxy using the following command:

```sh
ansible-galaxy install -p . geerlingguy.mysql
```

* Create an Nginx role with Ansible-Galaxy using the following command:

```sh
ansible-galaxy install -p . geerlingguy.nginx
```

* Create an Apache role with Ansible-Galaxy using the following command:

```sh
ansible-galaxy install -p . geerlingguy.apache
```

* Rename the role directories you downloaded.

```sh
mv geerlingguy.apache/ apache && mv geerlingguy.nginx/ nginx && mv geerlingguy.mysql/ mysql && ll
```

### Step 6: Merge changes from the roles-feature into the main branch

* Run the following command to view the untracked files (i.e. the roles directories you just created):

```sh
cd .. && git status
```

* Add the untracked files, commit the changes and push all the changes from the `roles-feature` branch to the `main` branch.

```sh
git add roles/apache/ roles/mysql/ roles/nginx/
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature
```
* This will prompt you to input your GitHub account username and password.

* Input your username and for the password you will need to input your Personal Access Token.

* The following steps are taken to setup a Personal Access Token:

1. Go to your GitHub Account, click on your Profile Icon and click on `settings`
2. Click on `Developer Settings`
3. Click on `Personal access tokens` and `Tokens (classic)`
4. Click on `Generate new token` and `Generate new token (classic)`
5. Give the Token a name of your choice (i.e. dynamic assignment project) and tick all the boxes.
6. Click on `Generate token`
7. Copy the token you just created and head back to the `Jenkins-Ansible` terminal.

* Paste the token in the prompt for the password of your GitHub account.

* Go to your `ansible-config-mgt` repository on GitHub and click on the `Compare & pull request` button.

* Click on the `Create pull request` button.

* Click on the `Merge pull request` button.

* Click on the `Confirm merge` button.

* Go to the `ansible-config-mgt` directory on your local machine and run the following command to pull the changes:

```sh
git pull
```

### Step 7: Create a `loadbalancer.yml` file in the static-assignments directory, update the `site.yml` and `dev.yml` file in the playbook and env-vars directories respectively

* Create a `loadbalancer.yml` file in the `static-assignments` directory.

```sh
cd static-assignments && touch loadbalancer.yml
```

* Paste the following codebase to reference your roles and conditions for running your roles in the `loadbalancer.yml` file:

```sh
- hosts: lb
  become: true
  roles:
    - { role: roles/nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: roles/apache, when: enable_apache_lb and load_balancer_is_required }
```

* Update the `site.yml` file to import the `loadbalacer.yml` file.

```sh
- name: Loadbalancer assignment
  hosts: lb
- import_playbook: ../static-assignments/loadbalancers.yml
  when: load_balancdr_is_required
```

* Declare variables in the `dev.yml` file in the `env-vars` directory to meet the conditions set to execute a role at a time.

```sh
enable_nginx_lb: true
enable_apache_lb: false
load_balancer_is_required: true
```

### Step 8: Update the `ansible-config-mgt` repository on GitHub with the latest configurations

* Run the following command to view the untracked files (i.e. the file created and modified files):

```sh
git status
```

* Add the untracked files and commit the changes.

```sh
git add ../env-vars/dev.yml ../playbooks/site.yml loadbalancers.yml && git commit -m "uploaded a configuration file and modified 2 configuration files"
```

* Push the changes to the `main` branch.

### Step 9: Run the Ansible Playbook with the Nginx role

Remember that a webhook was configured to save artifacts in the `ansible-config-artifact` directory on the `Jenkins-Ansible` server in the previous project. Hence, the latest files in the `ansible-config-mgt` repository on GitHub will be stored in the `ansible-config-artifact` directory.

* Go to the `ansible-config-artifact` directory on the `Jenkisn-Ansible` server.

```sh
cd /home/ubuntu/ansible-config-artifact/
```

![cd ansible-config-artifact](./images/9.%20cd%20:home:ubuntu:ansible-config-artifact.png)

* Run the Ansible Playbook.

```sh
ansible-playbook -i inventory/dev playbooks/site.yml
```

![ansible playbook nginx1](./images/9.%20ansible%20playbook%20nginx1.png)
![ansible playbook nginx2](./images/9.%20ansible%20playbook%20nginx2.png)

### Step 10:Run the Ansible Playbook with the Apache role

* Change the variables for enabling Apache and Nginx in the `dev.yml` file in the `env-vars` directory to meet the conditions for executing the Apache role.

```sh
enable_nginx_lb: false
enable_apache_lb: true
load_balancer_is_required: true
```

![dev.yml](./images/10.%20dev_yml.png)

* Run the Ansible Playbook.

```sh
ansible-playbook -i inventory/dev playbooks/site.yml
```

![ansible playbook error apache1](./images/10.%20ansible%20playbook%20error%20apache1.png)
![ansible playbook error apache2](./images/10.%20ansible%20playbook%20error%20apache2.png)

_Note that the **Ensure Apache has selected state and enabled on boot** task failed when the playbook ran. This is because Nginx and Apache can not run at the same time. To correct this, an `lb2` host will be introduced in the `dev` file in the `inventory` directory and the host on the `loadbalancer.yml` file will be changed to `lb2`_.

![inventory/dev](./images/10.%20inventory:dev.png)
_The updated `dev` file in the inventory directory_

![loadbalancer.yml](./images/10.%20loadbalancer_yml.png)
_The updated `loadbalancer.yml` in the `static-assignments` directory_

* Run the Ansible Playbook again.

```sh
ansible-playbook -i inventory/dev playbooks/site.yml
```

![ansible playbook apache1](./images/10.%20ansible%20playbook%20apache1.png)
![ansible playbook apache2](./images/10.%20ansible%20playbook%20apache2.png)

### Step 11: Run the Ansible Playbook with the MySQL role

* Change the variables for enabling Apache, Nginx and MySQL in the `dev.yml` file in the `env-vars` directory to meet the conditions for executing the MySQL role.

```sh
enable_nginx_lb: false
enable_apache_lb: false
enable_mysql_lb: true
load_balancer_is_required: true
```

![dev.yml](./images/11.%20dev_yml.png)

* Update the `loadbancer.yml` file in the `static-assignments` directory to include the MySQL role and change the host to `db`

```sh
- { role: roles/mysql, when: enable_mysql_lb and load_balancer_is_required }
```

![loadbalancer.yml](./images/11.%20loadbalancer_yml.png)

* Update the `site.yml` file in the `playbooks` directory to have a `db` host in the **Loadbalancers assignment** import.

![site.yml](./images/11.%20site_yml.png)

* Run the Ansible Playbook.

```sh
ansible-playbook -i inventory/dev playbooks/site.yml
```

![ansible playbook mysql1](./images/11.%20ansible%20playbook%20mysql1.png)
![ansible playbook mysql2](./images/11.%20ansible%20playbook%20mysql2.png)
![ansible playbook mysql3](./images/11.%20ansible%20playbook%20mysql3.png)