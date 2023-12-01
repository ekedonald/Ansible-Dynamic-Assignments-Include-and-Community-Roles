# Ansible Dynamic Assignments (Include) and Community Roles
The [Ansible Automation](https://github.com/ekedonald/Ansible-Automation-Project) and [Ansible Refactoring Assignments and Imports](https://github.com/ekedonald/Ansible-Refactoring-Assignments-and-Imports) projects have already explored how to perform configurations using `playbooks`, `roles` and `imports`. In this project, I will introduce [Dynamic Assignments](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse.html#includes-dynamic-re-use) by using the `include` module. From, the Ansible Refactoring Assignment project, it is evident that **Static Assignments** use the `import` module. However, the module that enables **Dynamic Assignments** is `include`.

When the `import` module us used, all statements are pre-processed at the time playbooks are parsed (i.e. when you execute `site.yml` playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements). This also means that during actual execution, if any statement changes, such statements will not be considered. Hence, it is **Static**.

On the other hand, when the `include` module is used, all the statements are processed only during execution of the playbook (i.e. after the statements are parsed, any changes to the statements encountered during execution will be used).

However, it is recommended to use **Static Assignments** for playbooks because it it more relaible while the **Dynamic Assignemnts** are difficult to debug due to its dynamic nature. Dynamic Assignments can be used for environment specific variables.

## How To Implement Ansible Dynamic Assignments (Include) and Community Roles
The following steps are taken to Implement Ansible Dynamic Assignments (Include) and Community Roles:

### Step 1: Introduce Dynamic Assignment into your File Structure

* Create and switch into the `dynamic-assignments` branch.

```sh
git checkout -b dynamic-assignments
```

* Create a `dynamic-assignments` directory and create an `env-vars.yml` file inside the directory.

```sh
mkdir dynamic-assignments && cd dynamic-assignmnets && touch env-vars.yml
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

1. The `include_vars` module was used instead of `include` because Ansible developers decided to seperate different features of the module. From Ansible version 2.8, the `include` module was depreciated and variants of `include_*` must be used (i.e. [include_role](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_role_module.html#include-role-module), [include_tasks](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_tasks_module.html#include-tasks-module) and [include_vars](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_vars_module.html#include-vars-module)).

2. We made use of the following [special variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html): `{{ playbook_dir }}` and `{{ inventory_file }}`.
The `{{ playbook_dir }}` helps Ansible determine the location of the running playbook and navigation to other paths on the filesystem while the `{{ inventory_file }}` will dynamically resolve the name of the inventory file being used then append `.yml` so that it picks up the required file within the `env-vars` directory.

3. The variables are included using the `with_first_found` loop which is used to loop through the list of files and the first one is used. This is good so that what we can always set default values in case an environment specific `env` file does not exist.

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

* Push all changes from the `dynamic-assignments` branch to the main branch.

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

### Step 6: Merge changes from the roles-feature into the main branch

### Step 7: Create a `loadbalancer.yml` file in the static-assignment directory, update the `site.yml` and `dev.yml` file in the playbook and env-vars directories respectively

### Step 8: Update the `ansible-config-mgt` repository on GitHub with the latest configurations

### Step 9: Run the Ansible Playbook with the Apache role

### Step 10:Run the Ansible Playbook with the Nginx role

### Step 11: Run the Ansible Playbook with the MySQL role
