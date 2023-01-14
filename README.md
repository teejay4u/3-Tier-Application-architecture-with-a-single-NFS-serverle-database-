>># Implentation of a 3-Tier-Application-architecture with a single NFS-server and database using AWS as cloud provider. 
## The following steps were followed in order to successfully implement this project:- 

1. Diagram of Implemented Project 
![](Diagram%20of%20Project.png)

2. Five Virtual machines were launched on AWS, 3 for webservers running linux redhat, one for NFS running linux redhat and one for the Database running Linux Ubuntu.  
![](1.%20Launching%20Servers%20For%20Project%20.png)

3. The next thing was to prepare the NFS server, three additional volumes were created in the same availability zone as the server and attached ready to be formatted. 
![](2.%20viewing%20available%20disks%20.png)

4. The drives were formatted using the command 
    - sudo gdisk /dev/xvdb 
    - sudo gdisk /dev/xvdc
    - sudo gdisk /dev/xvdd
![](3.%20Creating%20Partitions%20.png)

5. Partitions successfully Created 
![](4.%20created%20partitions%20.png)

6. Creating Physical Volumes using the command  
    - sudo pvcreate /dev/xvdb1
    - sudo pvcreate /dev/xvdc1
    - sudo pvcreate /dev/xvdd1

7. Creating Volume Group using the command : 
     sudo vgcreate webdata-vg /dev/xvdb1 /dev/xvdc1 /dev/xvdd1
![](6.%20Creating%20Volume%20Group%20.png)

8. Creating Logical Vloumes using the command :
    - sudo lvcreate -n lv-apps -L 9G webdata-vg
    - sudo lvcreate -n lv-logs -L 9G webdata-vg
    - sudo lvcreate -n lv-opt -L 9G webdata-vg
![](7.%20Creating%20Logical%20Volumes%20.png)

9. Mounting the directories with the command :
    - sudo mount /dev/webdata-vg/lv-apps /mnt/apps
    - sudo mount /dev/webdata-vg/lv-logs /mnt/logs

![](8.%20mounting%20directories%20.png) 

10. Setting Permissions for the directories created 

    - i.  sudo chown -R nobody: /mnt/apps
    - ii. sudo chown -R nobody: /mnt/logs
    - iii.sudo chown -R nobody: /mnt/opt

    - iv. sudo chmod -R 777 /mnt/apps
    - v.  sudo chmod -R 777 /mnt/logs
    - vi. sudo chmod -R 777 /mnt/opt

    - sudo systemctl restart nfs-server.service
![](9.%20setting%20permitions%20for%20directories%20to%20be%20used%20by%20webserver.png)

11. Configuring access to NFS for clients within the same subnet, by adding the subnet cidr of the webserver and populating it in the exports file :
     sudo vi /etc/exports
![](10.%20populating%20the%20exports%20file%20to%20allow%20ip%20cidr%20from%20webservers.png)

12. exporting the ports so its visible to NFS and checking ports running by NFS to be opened on the security group of the NFS server and allowing cidr subnet range of the webserver
     sudo exportfs -arv
     rpcinfo -p | grep nfs (view ports used by nfs)
     - open ports on security group of NFS server and allow only ip address cidr 
       range of webserver.
     - 2049 - TCP
     - 2049 - UDP
     - 111 -  TCP
     - 111  - UDP

![](11.%20exporting%20ports%20so%20webserver%20can%20reach%20nfs%20server.png)

13. Configuring the database and granting access to the created user and allowing only ip addresses from the webserver cidr subnet 
    - sudo mysql 
    - create database tooling;
    - create user '<user>'@'<webserver cidr range> identified by '<password>;
    - grant all on tooling.* to '<user>'@'<webserver cidr range>';
    - flush privileges;
    - edited the bind address in /etc/mysql to 0.0.0.0
![](12.%20Configuring%20the%20DB%20and%20granting%20access%20to%20webserver%20ip%20cidr%20.png)

14. Configuring webserver : 
    - Sudo yum update -y
    - installed NFS client with the command:
    - sudo yum install nfs-utils nfs4-acl-tools -y
    - Mounted /var/www/ and target the NFS server’s export for apps
    - sudo mkdir /var/www
    - sudo mount -t nfs -o rw,nosuid <NFS-Private-IP-Address>:/mnt/apps /var/www
    - Checked it was correctly mounted by running **df -h**
    - Making sure changes persist upon reboot by editing **sudo /etc/fstab** file 
    - adding the code on the webserver /sudo /etc/fstab :
    - <NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0
    - did same thing for the logs directory:
    - <NFS-Server-Private-IP-Address>:/mnt/logs /var/log/httpd nfs defaults 0 0

![](13.Mounting%20directory%20on%20webserver%20to%20nfs%20server%20using%20private%20ip%20of%20nfs%20server.png)

![](14.%20testing%20to%20see%20mount%20point%20is%20active%20on%20webserver%20.png)

![](18.%20Mounting%20the%20logs%20directory%20apache%20will%20use%20on%20the%20webserver%20to%20the%20nfs%20mount%20point.png)

15. Testing the mount points are working by creating a file hello.txt

![](15.%20testing%20the%20mount%20point%20by%20creating%20a%20file%20on%20webserver%20and%20confirming%20its%20on%20nfs%20server.png)

16. File replicated on NFS-server
![](16.%20File%20replicated%20on%20NFS-server.png)

17. Installing Apache and its dependencies 
    - sudo yum install httpd -y

    - sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest- 
      8.noarch.rpm

    - sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi- 
      release-8.rpm

    - sudo dnf module reset php

    - sudo dnf module enable php:remi-7.4

    - sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

    - sudo systemctl start php-fpm

    - sudo systemctl enable php-fpm

    - sudo setsebool -P httpd_execmem 1 
![](17.%20installing%20apache%20and%20its%20dependencies%20.png)

    - Forked the source code for the tooling app 
![](19.%20forking%20the%20application%20to%20webserver%20.png)

    - copied the html file to /var/www/html using sudo cp -R html/. /var/www/html

    - opened port 80 in security group of webserver 

    - checking permissions to the /var/www/html directory and also disabling    
      SELinux by running 
    - sudo setenforce 0
    - disabling setenforce so it remains disabled upon restart :
    - sudo vi /etc/sysconfig/selinux
    - setting setenforce to be disabled
![](20.%20setting%20permission%20of%20apache%20folder%20of%20setenforce%20to%200%20and%20disabling%20it%20to%20make%20the%20change%20persist.png)

18. updating functions.php file with DB-credentials 

![](21.%20Editing%20functions.php%20file%20with%20our%20dp%20credentials%20.png)

19. updating website configurations to connect to database
    - mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling- 
      db.sql
![](22.%20updating%20website%20configuration%20to%20connect%20to%20the%20database.png)

20. Login Page of Tooling App deployed 

![](23.%20Login%20page%20of%20tooling%20app.png)

21. Tooling web-App 

![](24.%20tooling%20page%20loaded%20successfully%20.png)
