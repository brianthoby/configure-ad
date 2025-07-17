<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial walks through the complete deployment and configuration of Active Directory Domain Services using virtual machines hosted in Microsoft Azure. It simulates an on-prem AD setup using a cloud environment to mirror real-world enterprise infrastructure..<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

1. Provision 2 Azure virtual machines on the same virtual network:
    - One for the Domain Controller (Windows Server 2022)
    - One for the Client (Windows 10)
2. Install and configure Active Directory Domain Services (AD DS) on the domain controller.
3. Join the client VM to the domain.
4. Create and manage OUs, users, and groups.
5. Use PowerShell scripting for bulk user creation.
6. Configure remote access and user policies.
7. Monitor user activity via Group Policy Management and Event Viewer.


<h2>Deployment and Configuration Steps</h2>

**STEP 1: Provision VMs in Azure**
![image](https://github.com/user-attachments/assets/f86e2c5e-d47e-46e0-8538-0212ec1da1dc)

- Create a resource group and within it 2 virtual machines within the same network and region. 
  - Domain Controller: Windows Server 2022, 2 vCPUs
  - Client: Windows 10 (21H2)
- Create a username and password then you are free to hit "review and create" to deploy the VMs.
- The picture above shows the Domain controller's vm setup

<img width="639" alt="DC connect" src="https://github.com/user-attachments/assets/cbb1ccff-2df1-4e68-b165-10460a75199c" />

- Now that we have our "computers" we need to login the Domain Controller to disable the firewalls so that we are able to test connectivity
- Log-in the domain controller through remote desktop protocol/remote desktop connection with the username and password you created when making the VM's
- We are here to disable the windows firewall so that we can test the connectivity from the end-user computer

**STEP 2: Disable Firewall on Domain Controller**
![image](https://github.com/user-attachments/assets/4ad96ab2-4a3a-4fb6-936c-d3025185515d)
- Once logged in, right click the windows start menu located in the bottom left corner and click run which will allow you to run a program by name
- Run wf.msc for windows defender firewall
- once the firewall is open, click the "windows Defender Firewall Properties" and disable the firewalls by changing the firewall state to off in the domain, private, and public profiles like in the picture above.

**STEP 3: Assign Static IP and set DNS**
![image](https://github.com/user-attachments/assets/9de984cb-a3cf-442f-b231-67c611f1f075)

- Within Azure configure the Domain Controller's NIC privite IP address to be static, for a consistent IP address
- Once it is static set the End-user's DNS settings to the Domain Controller's private IP address, so the end user can join the domain of our domain controller

**STEP 4: Test network connectivity**
![powershell con DNS](https://github.com/user-attachments/assets/675be111-7918-4854-8ace-f8f9725ba927)

- Log into the End-user computer with the username and password created
- Once in the end user's computer open powershell and ping the Domain Controller using it's IP address, you shoud get packets back as above showing connectivity
- Then we can run "IPconfig/ all" to confirm the output for the DNS settings, which should show the Domain Controller's private IP as in the above picture

**STEP 5: Install active directory services**
![image](https://github.com/user-attachments/assets/ea88dbf7-7aeb-4f09-8d45-17614920eae5)

- Go back into the Domain Controller and open service manager within the start menu
- Open "add roles and features" and click next until you get to server roles

![image](https://github.com/user-attachments/assets/f0320abb-9ec3-4207-8fd8-88345106947e)

- under server roles click "actve directory Domain services", a pop up will come up with the option to "add features". Click the "add features" button, then continue to click next until able to install.

**STEP 6: Promote to Domain Controller**
![promote](https://github.com/user-attachments/assets/c4eb48c1-9ec9-44fe-a889-f25bc2bf8c5f)
- After install, click the flag icon in the upper right corner
- Once clicked select the option to promote the server to a domain controller

![image](https://github.com/user-attachments/assets/41b6939d-4fa2-47e5-83c6-cef2755d3842)

- Promoting the server will redirect you to the configuration of the active directory domin services
- Add a new forest, and name your domain in this case it's "Mydomain.com"
- Next you will set your restore mode password, used to repair and/or restore the Active directory's database

![image](https://github.com/user-attachments/assets/d414ec8d-6e0e-43a7-84f1-9fe032274df5)

- Uncheck the option to create DNS delegation
- Click next until you make it to the prerequisites page
- Here you will install the active directory domain services
- Once installed, the VM will automatically restart
- afterwards sign-in again, but this time add your domain name and a backslash before your original username. This way you are able to specify that your logging in as a domain user
- New login will look like this "Mydomain.com\username"

**STEP 7: Create OU's & Users in AD**
![image](https://github.com/user-attachments/assets/9c515d95-6d2d-4477-8f65-b481d810d7d6)

- Go back into the Domain controller, in the start menu under windows Adminstative tools open "Active Directory Users and Computers"
- Right click the my domain.com file, then under new click Organizational Unit(OU) and name it as you please. I will make 2, 1 called ADMIN and another called EMPLOYEES.
- However in order to make it truly an admin we must add users to the domain admin security group

![image](https://github.com/user-attachments/assets/bbad8b33-7ad6-4483-aa23-b94750007f80)

- Firs, add a user into the ADMIN OU created
- Open the ADMIN OU and right click inside, under new click on user and fill in the user's requested information as seen in the image above
- Next, fill in the password and set the password's settings for the user

![add as admin](https://github.com/user-attachments/assets/5e91144f-644a-4490-b171-5c03408a64db)

- Now to make the user officially an admin right click the user and open properties
- in properties go to "Member Of", click add, type in doamin admins, click check names, and lastly hit okay to apply the changes
- now the Active Directory has been configured and has at least 1 admin user

**STEP 8: Join the end-user PC to the domain**
![image](https://github.com/user-attachments/assets/0ce9b6b4-e13b-41ed-a530-49bf78c38843)

- On the end-user VM right click the windows menu and open system
- In the right hand corner select "rename this PC (advanced)" which will open system properties, as shown in images above
- Under computer name select the option that says "change" which will allow us to join our domain
- inside the change option under "member of" will be the option to switch from the assigned workgroup to a domain, click domain and type in the domain name and click "OK"

![add domain to client](https://github.com/user-attachments/assets/bdd8fa08-8e26-4f10-baf5-e97c0b360e67)

- After clicking OK, you will be prompted to log into the domain with an account with permission to join.
- Log in the end-user's VM with the admin account created on our Domain Controller.

**STEP 9: Grant RDP access to domain users**
![rdp access](https://github.com/user-attachments/assets/bf3b5ef3-c5b7-478e-91ce-c1a1f4c5cf13)

- Now we can allow the users that we create to remotely sign in via the end-user VM
- open systems and go to remote desktop
- Under user accounts click "select users that can remotely access this PC"
- Click add and in the empty text box type in domain users, click check names, and okay.
- Now it is official, any account registered in the domain controller can sign in via the end-user computer

**STEP 10: Bulk user creation**
![image](https://github.com/user-attachments/assets/59df3f6d-602a-4b37-afaa-80b5b3af75e1)

- Now let's create more users by making several accounts in one go
- open Windows PowerShell ISE, make sure to run as an administrator
- hit file and save as to create a file for the users we will create, and save.
- create a script to create 10,000 users (or desired amount) with default passwords, assigning them to the Employee's OU

![image](https://github.com/user-attachments/assets/41791136-1785-4a35-b2b3-08fc71b8eac8)

- Here i ran a script to create 10,000 users all under the same password, "Password1".
- once the script is inputted hit the green triangle on the top toolbar to run the script, once it begins any red means there was an error
- Highlighted in the images above will show the OU I chose these accounts to get created in which is the EMPLOYEE folder created earlier
- you can check on these accounts by going back in "Active Directory Users and Computers" and opening the OU, ours being EMPLOYEE

**STEP 11: Log in with a domain user**
  ![image](https://github.com/user-attachments/assets/b9e63de8-2c0e-43c9-b058-2573b499757c)

- Now that our users are created lets log into one of those accounts to confirm
- Pick an account and log in to the end-user VM with those credentials as in the image above via RDP, using Remote Desktop Connection
- you can check all past users that logged in this computer going to local disk (C:) and opening the users file as seen above. Any account that logs in will be saved here

**STEP 12: Password Policy and Activity Monitoring**
![image](https://github.com/user-attachments/assets/ae21afd5-746c-481a-8371-c876ad8797a8)

- As far as managing accounts, 2 of the most common things will be password assistance and monitoring/tracing account activity
- In the Domain contrller within group policy management in the domain's folder you can add group policies to implement mass changes
- you can edit the group policy by clicking edit, under computer configuration drop down security settings and then account policies.
- There will be several options for editing domain policies
- The common passwords policies utilized refer to lockouts, which can be adjusted in account lockout policies
  - you may change the duration, waiting period, threshhold, and more.
- You use event viewr to monitor security logs as displayed in the image on the right. In event viewer open the windows log and click security to see all account activity like login attempts, lockouts, and policy violations.

**This project simulates:**
- Real-world Active Directory setup
- Remote user management
- Enterprise-level access control
- Automated account provisioning
- Activity monitoring & security hardening
*This concludes our comprehensive breakdown on configuring, deploying, and managing active directory.*
