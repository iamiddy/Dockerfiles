# Dockerfiles
Dockerfiles repo

### Data Volume Containers

-- If you have some persistent data that you want to share between containers, or want to use from non-persistent containers, it's best to create a named Data Volume Container, and then to mount the data from it. <br/>

Let's create a new named container with a volume to share. While this container doesn't run an application, it reuses the ubuntu:14.10 image so that all containers are using layers in common, saving disk space. <br/>

<tt> $ sudo docker create -v /dbdata --name dbdata ubuntu:14.10 /bin/true  </tt> --- create volume container <br/>

You can then use the --volumes-from flag to mount the /dbdata volume in another container.

<tt> $ sudo docker run -d --volumes-from dbdata --name db1 mongo </tt> <br/>

And another:

<tt> $ sudo docker run -d --volumes-from dbdata --name db2 mongo </tt> <br/>

In this case, if the mongo image contained a directory called /dbdata then mounting the volumes from the dbdata container hides the /dbdata files from the mongo image. The result is only the files from the dbdata container are visible.

You can use multiple --volumes-from parameters to bring together multiple data volumes from multiple containers.

If you remove containers that mount volumes, including the initial dbdata container, or the subsequent containers db1 and db2, the volumes will not be deleted. To delete the volume from disk, you must explicitly call docker rm -v against the last container with a reference to the volume. This allows you to upgrade, or effectively migrate data volumes between containers.

##Backup, restore, or migrate data volumes

---- Another useful function we can perform with volumes is use them for backups, restores or migrations. We do this by using the --volumes-from flag to create a new container that mounts that volume, like so:

<tt> $ sudo docker run --volumes-from dbdata -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata </tt> <br/>

Here we've launched a new container and mounted the volume from the dbdata container. We've then mounted a local host directory as /backup. Finally, we've passed a command that uses tar to backup the contents of the dbdata volume to a backup.tar file inside our /backup directory. When the command completes and the container stops we'll be left with a backup of our dbdata volume. <br/>

You could then restore it to the same container, or another that you've made elsewhere. Create a new container.
<tt> $ sudo docker run -v /dbdata --name dbdata2 ubuntu /bin/bash </tt> <br/>

Then un-tar the backup file in the new container's data volume. <br/>
<tt>$ sudo docker run --volumes-from dbdata2 -v $(pwd):/backup busybox tar xvf /backup/backup.tar </tt>

You can use the techniques above to automate backup, migration and restore testing using your preferred tools


<tt> docker run -d -p 27017:27017 -p 8080:8080 --volumes-from dbdata --name sboot iamiddy/springboot </tt>

Step : List the /data/db directory

<tt> docker run --volumes-from dbdata busybox ls -al /data/db </tt>



### Workarounds
Note: The following steps are meant as a temporary solution and won't be needed anymore in the future.

#### Port forwarding

Let's say your Docker container exposes the port 8000 and you want access it from your other computers on your LAN. You can do it temporarily, using ssh: <br/>

Run following command (and keep it open):

<tt> $ boot2docker ssh -vnNTL 8000:localhost:8000 </tt> <br/>
or you can set up a permanent VirtualBox NAT Port forwarding:

<tt> $ VBoxManage modifyvm "boot2docker-vm" --natpf1 "tcp-port8000,tcp,,8000,,8000"; </tt> <br/>
If the vm is already running, you should run this other command:

<tt> $ VBoxManage controlvm "boot2docker-vm" natpf1 "tcp-port8000,tcp,,8000,,8000"; </tt> <br/>
Now you can access your container from your host machine under localhost:8000. <br/>

Copy existing folder from host to docker volume container. <br/>

First of all the credit must be given to the person who has this covered already on his blog here. but I thought it is worth mentioning it again here because we are covering the topic of backup and restore.<br/>

Step 1. Create volume only docker container named as testdata.

1 <tt> docker run -v /test/data --name testdata busybox true </tt>
Step 2. Copy the folder named as sample from current host directory to volume container.

1 <tt> tar -c sample/ | docker run -i --rm -w /test/data --volumes-from testdata busybox tar -xv </tt>
 
 The tar command  pipes the content of sample folder into the docker volume container directory [/test/data].
You can use the ls command like we did earlier change the mongodata to testdata and [/data/db] to [/test/data]
All the contents from sample directory copied will be listed.


### Mongo data container -- dbdata
mongo image mongo 2.6.10 <br/>
create mongo db container <br/>
<tt>sudo docker run -i -t -p 27027:27017 -p 28027:28017 --name mongodb  --volume-from dbdata mongo </tt>
