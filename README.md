# Implentation of a 3-Tier-Application-architecture with a single NFS-server and database using AWS as cloud provider. 
## The following steps were followed in order to successfully implement this project:- 

1. Diagram of Implemented Project 
![](Diagram%20of%20Project.png)

2. Five Virtual machines were launched on AWS, 3 for webservers running linux redhat, one for NFS running linux redhat and one for the Database running Linux Ubuntu.  
![](1.%20Launching%20Servers%20For%20Project%20.png)

3. The next thing was to prepare the NFS server, three additional volumes were created in the same availability zone as the server and attached ready to be formatted. 
![](2.%20viewing%20available%20disks%20.png)

4. The drives were formatted using the command 
    sudo gdisk /dev/xvdb 
    sudo gdisk /dev/xvdc
    sudo gdisk /dev/xvdd
![](3.%20Creating%20Partitions%20.png)

5. Partitions successfully Created 
![](4.%20created%20partitions%20.png)

6. Creating Physical Volumes using the command  
     sudo pvcreate /dev/xvdb1
     sudo pvcreate /dev/xvdc1
     sudo pvcreate /dev/xvdd1

7. Creating Volume Group using the command : 
     sudo vgcreate webdata-vg /dev/xvdb1 /dev/xvdc1 /dev/xvdd1
![](6.%20Creating%20Volume%20Group%20.png)

8. Creating Logical Vloumes using the command :
     sudo lvcreate -n lv-apps -L 9G webdata-vg
     sudo lvcreate -n lv-logs -L 9G webdata-vg
     sudo lvcreate -n lv-opt -L 9G webdata-vg
![](7.%20Creating%20Logical%20Volumes%20.png)

9. Mounting the directories with the command :
     sudo mount /dev/webdata-vg/lv-apps /mnt/apps
     sudo mount /dev/webdata-vg/lv-logs /mnt/logs
![](8.%20mounting%20directories%20.png) 

10. Setting Permissions for the directories created 
     sudo chown -R nobody: /mnt/apps
     sudo chown -R nobody: /mnt/logs
     sudo chown -R nobody: /mnt/opt

     sudo chmod -R 777 /mnt/apps
     sudo chmod -R 777 /mnt/logs
     sudo chmod -R 777 /mnt/opt

     sudo systemctl restart nfs-server.service
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
![](12.%20Configuring%20the%20DB%20and%20granting%20access%20to%20webserver%20ip%20cidr%20.png)

14. 