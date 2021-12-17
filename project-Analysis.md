## PROJECT 9
##### TOOLING WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION. INTRODUCTION TO JENKINS

In the last Project 8 we introduced horizontal scalability concept, which allow us to add new Web Servers to the Tooling Website and i have successfully
deployed a set up with 2 Web Servers and a Load Balancer to distribute traffic between them. Since it was just two or three servers – it was not a big dea
to configure them manually. But imagine that i would need to repeat the same task repeatedly adding dozens or even hundreds of servers.
However, DevOps is about Agility, and speedy release of software and web solutions. One of the ways to guarantee fast and repeatable deployments is 
Automation of routine tasks.
In this project I am going to utilize Jenkins Continuous integration CI capabilities to make sure that every change made to the source
code in GitHub https://github.com/clement2019/tooling will automatically be updated to the Tooling Website, Task
I am to enhance the architecture prepared in Project 8 by adding a Jenkins server, configure a job to automatically deploy source codes changes
from Git to NFS server.
Here is how i updated architecture will look like upon completion of this project:

![image](https://user-images.githubusercontent.com/55473846/146602683-f72d2908-c7f0-45b3-90f2-be26447962f6.png)

INSTALL AND CONFIGURE JENKINS SERVER
Step 1 – Install Jenkins server
1.	I created an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"
2.	Connected to the server using ssh connection with gitbash
3.	Install JDK (since Jenkins is a Java-based application)
sudo apt update		

![image](https://user-images.githubusercontent.com/55473846/146602865-eb9c0652-b102-47b0-a307-8e2d2470d5fd.png)


sudo apt install default-jdk-headless

![image](https://user-images.githubusercontent.com/55473846/146602995-8c8c7fdc-8f85-4b8b-a411-aa462d8a4c4a.png)

3.	Install Jenkins
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins


![image](https://user-images.githubusercontent.com/55473846/146603086-b1e67b1a-b5ff-4a01-a64f-5d12d05f6370.png)

Now to be sure Jenkins is up and running I ran the command below
sudo systemctl status Jenkins

![image](https://user-images.githubusercontent.com/55473846/146603236-1f395eb2-a30f-48bf-93f6-13b891644eaa.png)

By default Jenkins server uses TCP port 8080 – I opened it by creating a new Inbound Rule in myEC2 Security Group

![image](https://user-images.githubusercontent.com/55473846/146603327-88824bc9-a8fd-4797-a193-e1bec66cafc6.png)

Perform initial Jenkins’s setup.

From my browser I could access 
http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080	
http:// 18.133.119.33:8080

I retrieved it from your server:
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

  ![image](https://user-images.githubusercontent.com/55473846/146603552-ba22a3e9-59fc-4a8f-9740-0ebb3abab195.png)

  ![image](https://user-images.githubusercontent.com/55473846/146603604-ca01c86e-612c-462d-88f1-02c92b595351.png)

  ![image](https://user-images.githubusercontent.com/55473846/146603649-8aeb28f4-1793-474c-8242-7c28b739c4bc.png)

  ![image](https://user-images.githubusercontent.com/55473846/146603699-54008f67-87dc-4e61-b6c6-e2921c2f4b2a.png)
  
Configure Jenkins to retrieve source codes from GitHub using Webhooks
  
In this part, I learned how to configure a simple Jenkins job/project (these two terms can be used interchangeably).
  This job was triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.
  
  ![image](https://user-images.githubusercontent.com/55473846/146603834-45687bc3-4c19-44f1-8fd2-e94f96bee890.png)

  ![image](https://user-images.githubusercontent.com/55473846/146603905-e07fb4b2-7f6d-4db9-aeb6-dc8bbe9d6e4b.png)

  ![image](https://user-images.githubusercontent.com/55473846/146603995-752c5118-387f-49bc-a2ef-71f540700b8f.png)
  
Go to Jenkins web console, click "New Item" and create a "Freestyle project"
  
  ![image](https://user-images.githubusercontent.com/55473846/146604205-22a679f3-59bd-43a8-a726-6a6f44ad745d.png)


I just opened the build and check in "Console Output" if it has run successfully.
If so – congratulations! You have just made your very first Jenkins build!
To now Click "Configure" my job/project and add these two configurations
Configure triggering the job from GitHub webhook:

  
Enable webhooks in your GitHub repository settings

  ![image](https://user-images.githubusercontent.com/55473846/146604288-4bb0e3ac-5232-48fb-ae4d-7781708de44d.png)
  
  ![image](https://user-images.githubusercontent.com/55473846/146604355-912288e0-1479-4540-ad96-689a2dde92e1.png)

  ![image](https://user-images.githubusercontent.com/55473846/146604431-4c89509a-4666-4cdb-a23d-bda4bcc79cd8.png)

  ![image](https://user-images.githubusercontent.com/55473846/146604533-351750bf-3bd7-4d5e-a08f-ffc48c157c86.png)

  ![image](https://user-images.githubusercontent.com/55473846/146604574-acdcc801-c30b-412b-ab23-309539f408d4.png)
  
  I have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is
  
  considered as ‘push’ because the changes are being ‘pushed’ and files transfer is initiated by GitHub). 
  
  There are also other methods: trigger one job (downstream) from another (upstream), poll GitHub periodically and others.As shown below

![image](https://user-images.githubusercontent.com/55473846/146604844-5acd7d48-23f3-4030-9276-8c825039f807.png)

  ![image](https://user-images.githubusercontent.com/55473846/146604928-f230d756-d5e7-4224-9035-49ad085173a3.png)
  
  ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/


![image](https://user-images.githubusercontent.com/55473846/146605117-23954858-72a5-43f1-9703-10bdf0a96343.png)

  CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH
  
I Configured Jenkins to copy files to NFS server via SSH
Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.
Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called "Publish Over SSH".
Install "Publish Over SSH" plugin.
On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item.
On "Available" tab search for "Publish Over SSH" plugin and install it
  
  ![image](https://user-images.githubusercontent.com/55473846/146605292-1da51e48-ccb6-4b04-8beb-7c0c03400329.png)
  
  ![image](https://user-images.githubusercontent.com/55473846/146605338-943c7765-8532-4229-8396-653e1cf5d168.png)
  
![image](https://user-images.githubusercontent.com/55473846/146605422-2b6fe79d-1fe3-4499-8b11-99d95d297b98.png)
  
  Configure the job/project to copy artifacts over to NFS server.
On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.
Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:
1.	Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
2.	Arbitrary name
3.	Hostname – can be private IP address of your NFS server
4.	Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
5.	Remote directory – /mnt/apps since our Web Servers use it as a mounting point to retrieve files from the NFS server
Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.

  ![image](https://user-images.githubusercontent.com/55473846/146605597-8dfc44ba-31e6-4f07-80ce-31aaaacdf1a0.png)

  
  ![image](https://user-images.githubusercontent.com/55473846/146605740-75c4daf8-240e-4c78-8248-b08b9788d090.png)

  Configure it to send all files produced by the build into our previously define remote directory. 
  In our case we want to copy all files and directories – so we use **.
If you want to apply some particular pattern to define which files to send – use this syntax.
  
  ![image](https://user-images.githubusercontent.com/55473846/146605820-2493106a-a97a-487e-9f66-dcac87defb4e.png)

  ![image](https://user-images.githubusercontent.com/55473846/146605977-66c8bb2f-7c17-4334-bcb9-b63eb2a75bb1.png)

  I encountered an error here and the error details can be seen in the console output of the Jenkins as show in the screenshots below  
  
  ”Error permission Denied”
  
  Now when I ran the persimmon command 
sudo chmond -R 777  /mnt
as shown below the error was resolved

  ![image](https://user-images.githubusercontent.com/55473846/146606201-0f2bc2b9-1e9b-443e-b742-ff1b65cb145c.png)

  ![image](https://user-images.githubusercontent.com/55473846/146606265-034e4e80-5c99-42a3-9c65-7660b34292f3.png)
  
Now Resolved
After I login toi the NFS! Server through ssh using my gitbash
/mns/apps
When i now ran the command below on the NFS! terminal
sudo chmod -R 777 /mnt
sudo chown -R /mnt
I saved this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.
Webhook triggered a new job and in the "Console Output" of the job you will find something like this:
SSH: Transferred 25 file(s)
Finished: SUCCESS
  
  ![image](https://user-images.githubusercontent.com/55473846/146606352-0bb397d7-48af-4f9c-9ca7-5f008537bfd5.png)
  
  ![image](https://user-images.githubusercontent.com/55473846/146606395-2b984089-cd1d-4359-8a5f-7a6be82833c5.png)
  
  To make sure that the files in /mnt/apps have been updated – connect via SSH using gitbash to the NFS1 server and check README.MD file
By running the command below
cat /mnt/apps/README.md
  
  ![image](https://user-images.githubusercontent.com/55473846/146606536-62e1f299-5724-435e-8ec2-6f73caf48fd2.png)
  
  ![image](https://user-images.githubusercontent.com/55473846/146606079-b8b13ecb-fd33-4e9d-ac31-1aac12afe0d5.png)


  Save the configuration, open your Jenkins job/project configuration page, and add another one "Post-build Action"


![image](https://user-images.githubusercontent.com/55473846/146606708-4462c745-b749-4b05-a5ea-8a6b6cc56b22.png)
  
  We can see from above that the content added to REAME.md is show meaning we have have been able to transfer the content of the file to our NFS1 server
Now I can see the changes, i had previously made in your GitHub – the job works as expected. This shows that I have successfully done a
  continuous integration CI of GIHub changes into the NFS server using  Jenkins 




  
