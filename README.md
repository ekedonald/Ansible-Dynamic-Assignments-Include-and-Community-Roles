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

* 

### Step 2: Update `site.yml` with Dynamic Assignments

### Step 3: Merge the changes from the dynamic-assignments branch into the main branch

### Step 4: SSH into the Jenkins-Ansible server, pull files from the `ansible-config-mgt` repository and create a `roles-feature` branch

### Step 5: Using Ansible-Galaxy, Download Community Roles for Apache, Nginx and MySQL into the roles directory

### Step 6: Merge changes from the roles-feature into the main branch

### Step 7: Create a `loadbalancer.yml` file in the static-assignment directory, update the `site.yml` and `dev.yml` file in the playbook and env-vars directories respectively

### Step 8: Update the `ansible-config-mgt` repository on GitHub with the latest configurations

### Step 9: Run the Ansible Playbook with the Apache role

### Step 10:Run the Ansible Playbook with the Nginx role

### Step 11: Run the Ansible Playbook with the MySQL role
