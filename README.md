# Ansible-Refactoring-and-static-assignment

### Ansible Refactoring and Static Assignment Project

In this project, we will improve the existing `ansible-config-mgt` repository by refactoring Ansible code, creating static assignments, and learning how to use the **import** functionality. This will enhance code reusability, organization, and efficiency.

---

### **Step 1: Jenkins Job Enhancement**

We need to improve the Jenkins job setup by saving build artifacts in a central location rather than creating a new directory for every build.

#### **1.1 Create Artifact Directory**
Log in to your Jenkins-Ansible server and create a directory to store artifacts after each build.

```
sudo mkdir /home/ubuntu/ansible-config-artifact
sudo chmod -R 777 /home/ubuntu/ansible-config-artifact
```
![image](https://github.com/user-attachments/assets/f2e6c53a-a6f8-4ac6-b9dd-054e86391653)


#### **1.2 Install Copy Artifact Plugin**
1. Open Jenkins -> Manage Jenkins -> Manage Plugins.
2. Under the **Available** tab, search for `Copy Artifact` and install it without restarting Jenkins.
![image](https://github.com/user-attachments/assets/6e40bb3e-2533-434c-9c53-a8c08a32f89e)

#### **1.3 Create Jenkins Freestyle Project**
- Go to Jenkins dashboard.
- Create a new Freestyle project named **save_artifacts**.
- Configure this project to trigger after the existing `ansible` project.
- The job should copy all artifacts to `/home/ubuntu/ansible-config-artifact`.
![image](https://github.com/user-attachments/assets/07761d0e-1c3e-49c3-b074-489d5486b8a6)
![image](https://github.com/user-attachments/assets/b8d2ed5d-783a-431a-9906-4a599ce3cb77)


#### **1.4 Test the Setup**
- Make a small change to the `README.md` in the `ansible-config-mgt` repository (e.g., in the master branch).
- Trigger the pipeline. Both Jenkins jobs should run, and the files should appear in `/home/ubuntu/ansible-config-artifact`.
![image](https://github.com/user-attachments/assets/db770475-4197-4bd6-b69c-057b8b18c3c6)
![image](https://github.com/user-attachments/assets/90a1c1bd-03cd-4acf-b7d9-b10090f4233c)


---

### **Step 2: Refactor Ansible Code by Using Imports**

To improve code organization, we'll refactor the Ansible code using imports.

#### **2.1 Create a New Branch**
Create a new branch for refactoring.

```
cd /home/ubuntu/ansible-config-mgt/
git branch refactor
git checkout refactor
```

#### **2.2 Create a Site Playbook**
Within the `playbooks` folder, create a new file named `site.yml`:

```
cd playbooks
touch site.yml
```

#### **2.3 Create Static Assignments Folder**
In the repository's root directory, create a folder called `staticassignments`:

```
cd ..
mkdir staticassignments
```

#### **2.4 Move and Import Existing Playbook**
Move the existing `common.yml` into the `staticassignments` folder and update `site.yml` to import it:

```
mv playbooks/common.yml staticassignments/
```
![image](https://github.com/user-attachments/assets/8d043fbd-d7ce-4c04-9871-934ae1a8a00e)


In `site.yml`:

```
---
hosts: all
import_playbook: ../staticassignments/common.yml
```
retry the ansible command 
```
ansible-playbook -i inventory/dev.yml playbooks/site.yaml
```
![image](https://github.com/user-attachments/assets/7bc53691-b7bc-4f1d-8d99-f8eb17c9ebf3)


#### **2.5 Create a New Playbook for Wireshark Deletion**
Create a new playbook `common-del.yml` inside `staticassignments`:

```
cd staticassignments
touch common-del.yml
```

Add the following content to delete Wireshark:

```
---
- name: Delete Wireshark on web, NFS, and DB servers
  hosts: webservers, nfs, db
  become: true
  tasks:
    - name: Remove Wireshark
      yum:
        name: wireshark
        state: removed

- name: Delete Wireshark on LB servers
  hosts: lb
  become: true
  tasks:
    - name: Remove Wireshark
      apt:
        name: wireshark-qt
        state: absent
        autoremove: yes
        purge: yes
        autoclean: yes
```
![image](https://github.com/user-attachments/assets/2a3ed48b-bbba-42aa-a318-d141bc0122bf)

#### **2.6 Update Site.yml**
Update `site.yml` to import `common-del.yml`:

```
---
hosts: all
import_playbook: ../staticassignments/common-del.yml
```

#### **2.7 Run the Playbook**
Test by running the playbook on your development environment:

```
ansible-playbook -i inventory/dev.yml playbooks/site.yml
```
![image](https://github.com/user-attachments/assets/a21b2b75-db43-4692-99d0-1406660d28a3)

---

### **Step 3: Configure UAT Web Servers with a Webserver Role**

Now, we will set up two UAT servers and configure them with the `webserver` role.

#### **3.1 Launch UAT EC2 Instances**
- Launch two EC2 instances using the RHEL 8 image and name them `Web1-UAT` and `Web2-UAT`.
![image](https://github.com/user-attachments/assets/6d4b1e6a-47bb-4d31-b1b8-f08638cb96f5)

#### **3.2 Create a Role for Webservers**
In your `ansible-config-mgt` repository, create a `roles` directory and generate a `webserver` role:

```
mkdir -p roles/webserver
cd roles/webserver
ansible-galaxy init webserver
```
![image](https://github.com/user-attachments/assets/2fdcab67-2189-482b-bf1f-f44857ae4f78)


You can also manually create the folder structure if preferred.

#### **3.3 Configure the Webserver Role**
In the `tasks/main.yml` file of the `webserver` role, add the following tasks to install Apache, clone the Tooling repository, and start the Apache service:

```
---
- name: Setup Apache and deploy a website
  hosts: all
  become: true
  tasks:
    - name: Install Apache
      yum:
        name: httpd
        state: present

    - name: Install Git
      yum:
        name: git
        state: present

    - name: Clone Tooling website
      git:
        repo: https://github.com/highbee2810/tooling.git
        dest: /var/www/html
        force: yes

    - name: Copy website content
      command: cp -r /var/www/html/html/ /var/www/

    - name: Start Apache service
      service:
        name: httpd
        state: started
```
![image](https://github.com/user-attachments/assets/dbf88e76-2e88-4802-b3ed-b6c268e64870)

#### **3.4 Update UAT Inventory**
Update the `uat.yml` inventory file with the UAT server IP addresses:

```
[uat-webservers]
<Web1-UAT-Private-IP> ansible_ssh_user='ec2-user'
<Web2-UAT-Private-IP> ansible_ssh_user='ec2-user'
```

#### **3.5 Configure Role Path in Ansible Config**
Ensure the role path is set in `ansible.cfg`:

```
roles_path = /home/ubuntu/ansible-config-mgt/roles
```

---

### **Step 4: Reference the Webserver Role**

Create a new playbook in the `staticassignments` folder for UAT webservers:

```
touch staticassignments/uat-webservers.yml
```

Add the following content to reference the `webserver` role:

```
---
- hosts: uat-webservers
  roles:
    - webserver
```
![image](https://github.com/user-attachments/assets/b0dcf780-9fd9-441a-b0d7-087eed0deafe)

Update `site.yml` to reference this new playbook:

```
---
hosts: all
import_playbook: ../staticassignments/common.yml

hosts: uat-webservers
import_playbook: ../staticassignments/uat-webservers.yml
```
![image](https://github.com/user-attachments/assets/a932a7aa-6cbc-4673-9d17-9eb7b98fd978)

---

### **Step 5: Commit & Test**

#### **5.1 Commit Changes**
Commit the changes, create a pull request, and merge into the master branch.

```
git add .
git commit -m "Refactor and configure UAT webservers"
git push origin refactor
```
![image](https://github.com/user-attachments/assets/9cf08209-1940-41f5-aedc-a9be11bc481b)


#### **5.2 Run the Playbook**
Run the playbook against your UAT environment:

```
ansible-playbook -i inventory/uat.yml playbooks/site.yml
```
![image](https://github.com/user-attachments/assets/04220e4b-8cc6-4aef-83de-cc171721d6d3)


#### **5.3 Verify Setup**
Access the UAT webservers via browser to verify:

```
http://<Web1-UAT-Public-IP>/index.php
```
![image](https://github.com/user-attachments/assets/f284d8a6-fe16-48e4-ba1a-d560790d8467)


Our Ansible architecture now looks like this:

![image](https://github.com/user-attachments/assets/93148d3c-0603-4c8b-9025-d9b06952da9b)


