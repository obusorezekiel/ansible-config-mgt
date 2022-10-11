## INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE

- Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.

![Screenshot from 2022-10-11 13-00-48](https://user-images.githubusercontent.com/23356682/195110719-a3b35d5b-9e99-41b3-aac3-3fce4f1b8ed2.png)

- In your GitHub account create a new repository and name it ansible-config-mgt.

- Instal Ansible

```
 sudo apt update
 sudo apt install ansible
```

- Configure Jenkins build job to save your repository content every time you change it – this will solidify your Jenkins configuration skills acquired in Project 9.

- Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository.

- Configure Webhook in GitHub and set webhook to trigger ansible build.

- Configure a Post-build job to save all (**) files, like you did it in Project 9.

- Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts)     in following folder

   `ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`
   

![Screenshot from 2022-10-11 13-10-56](https://user-images.githubusercontent.com/23356682/195110735-91f5ab08-f8f4-4d46-9874-b3f6fafcf8fb.png)

on the first build, your Jenkins config will look like this
![Screenshot from 2022-10-11 13-48-03](https://user-images.githubusercontent.com/23356682/195110741-09fd98a5-487b-4d07-b5e1-cc4c46f5d905.png)


   
 ## Step 2 – Prepare your development environment using Visual Studio Code
 
 - First part of ‘DevOps’ is ‘Dev’, which means you will require to write some codes and you shall have proper tools that will make your coding and debugging              comfortable – you need an Integrated development environment (IDE) or Source-code Editor. There is a plethora of different IDEs and Source-code Editors for            different languages with their own advantages and drawbacks, you can choose whichever you are comfortable with, but we recommend one free and universal editor that    will fully satisfy your needs – Visual Studio Code (VSC), you can get it here.

- After you have successfully installed VSC, configure it to connect to your newly created GitHub repository.

- Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance

  `git clone <ansible-config-mgt repo link>`
  
  
  ##  BEGIN ANSIBLE DEVELOPMENT
  
  - In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.
  Tip: Give your branches descriptive and comprehensive names, for example, if you use Jira or Trello as a project management tool – include ticket number (e.g. PRJ-     145) in the name of your branch and add a topic and a brief description what this branch is about – a bugfix, hotfix, feature, release (e.g. feature/prj-145-lvm)

- Checkout the newly created feature branch to your local machine and start building your code and directory structure

- Create a directory and name it playbooks – it will be used to store all your playbook files.

- Create a directory and name it inventory – it will be used to keep your hosts organised.

- Within the playbooks folder, create your first playbook, and name it common.yml

- Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod             respectively.

 - Step 4 – Set up an Ansible Inventory
     An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since our intention is to execute        Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our      hosts in such an Inventory.

     Save below inventory structure in the inventory/dev file to start configuring your development servers. Ensure to replace the IP addresses according to your own        setup.

    Note: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host – for this you can implement the concept of       ssh-agent. Now you need to import your key into ssh-agent:
    
    To learn how to setup SSH agent and connect VS Code to your Jenkins-Ansible instance, please see this video:

    - For Windows users – ssh-agent on windows
    - For Linux users – ssh-agent on linux


      ```
      
         eval `ssh-agent -s`
         ssh-add <path-to-private-key>
         
      ```
      
      
      Confirm the key has been added with the command below, you should see the name of your key

      `ssh-add -l`
      
      Now, ssh into your Jenkins-Ansible server using ssh-agent

      `ssh -A ubuntu@public-ip`
      
      Also notice, that your Load Balancer user is ubuntu and user for RHEL-based servers is ec2-user.

      Update your inventory/dev.yml file with this snippet of code:
      
      ```
      
            [nfs]
        <NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'
            [webservers]
        <Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
        <Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'
            [db]
        <Database-Private-IP-Address> ansible_ssh_user='ec2-user' 
            [lb]
        <Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'
        
        ```
        
        ![Screenshot from 2022-10-11 14-57-35](https://user-images.githubusercontent.com/23356682/195111521-fb7b93c7-03a0-4272-b0b8-f09d1fb43f04.png)


        
   ## CREATE A COMMON PLAYBOOK
   
   - Step 5 – Create a Common Playbook
     It is time to start giving Ansible the instructions on what you needs to be performed on all servers listed in inventory/dev.

   - In common.yml playbook you will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

   - Update your playbooks/common.yml file with following code:


    ```
    
           ---
           - name: update web, nfs and db servers
             hosts: webservers, nfs, db
             remote_user: ec2-user
             become: yes
             become_user: root
             tasks:
               - name: ensure wireshark is at the latest version
                 yum:
                 name: wireshark
                 state: latest
            - name: update LB server
              hosts: lb
              remote_user: ubuntu
              become: yes
              become_user: root
              tasks:
                - name: Update apt repo
                  apt: 
                  update_cache: yes
                 - name: ensure wireshark is at the latest version
                   apt:
                     name: wireshark
                     state: latest
                     
       ```
          

          
    - use git commands to add, commit and push your branch to GitHub.

       ```
         git status
         git add <selected files>
         git commit -m "commit message"
         
        ```
        
    - Create a Pull request (PR)

    - Wear a hat of another developer for a second, and act as a reviewer.

    - If the reviewer is happy with your new feature development, merge the code to the master branch.

    - Head back on your terminal, checkout from the feature branch into the master, and pull down the latest changes.

     
   - Once your code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to                                                   `/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/` directory on Jenkins-Ansible server



    ## RUN FIRST ANSIBLE TEST
    
   
    
        cd ansible-config-mgt
        ansible-playbook -i inventory/dev.yml playbooks/common.yml
        
   ```
![Screenshot from 2022-10-11 14-57-57](https://user-images.githubusercontent.com/23356682/195112594-033b4efc-b66b-48ea-9bf8-7ab602609686.png)
   ![Screenshot from 2022-10-11 14-01-33](https://user-images.githubusercontent.com/23356682/195112563-bf3e5234-88d7-4c1e-a4fa-ff33e8df4f3f.png)
![Screenshot from 2022-10-11 14-02-30](https://user-images.githubusercontent.com/23356682/195112576-8bf4b642-7bcd-4257-8e28-644d0733bc44.png)
![Screenshot from 2022-10-11 14-04-28](https://user-images.githubusercontent.com/23356682/195112584-794717b0-5825-4b91-9e9a-00ec5c74acf3.png)


