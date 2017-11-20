# Setting Up A Two Machine Spark Cluster
## Ubuntu Guest (VirtualBox) on a Windows host

Lately I took up a project which required a sample 2-machine spark cluster. Having two laptops at my hostel room, inspired me to set it up myself.
Setting up a cluster on Windows seemed unlikely, although I had found a blog on that too. The path I chose was to set it up on Ubuntu installed as VirtualBox guests on two Windows machines.

Being a novice that I am, I had little idea about how to setup a cluster. The little I knew came from my experience at a consulting giant where I had worked in a project which involved setting up a 2-machine Hadoop cluster.

While in the process, I faced a lot of hick ups due to my ignorance of some issues and the tutorials available assuming that the reader would be aware of those issues.

Hence I write this blog for novices like me to `Setup a 2-machine Apache Spark cluster from scratch`. I have learned all the concepts from different tutorials and they are mentioned as references at the last.

### Pre-requisites:

* A little VirtualBox thing
* SSH concepts
* root access on the Linux virtual machines
* basic understanding of cluster terms such as Master, Slave, SSH
* Internet connectivity :P

### Step by Step:

1. Create a virtual machine using VirtualBox with Ubuntu (I chose 16.04 because that version is officially supported for Ambari server also) - might want to refer [this Q&A](https://askubuntu.com/questions/142549/how-to-install-ubuntu-on-virtualbox) .
2. Configure your virtual machine's networking mode to `Bridged Adapter`. This is an **important** step. It had me stuck for a while. This option in VirtualBox allows your virtual machine to have its own IP address and enable you to make password less SSH to them. This [stackExchange question](https://superuser.com/questions/119732/how-to-do-networking-between-virtual-machines-in-virtualbox) introduced me to the networking modes and finally I learned about them on [VirtualBox Networking Guide](http://www.virtualbox.org/manual/ch06.html#network_bridged).
3. After setting up your Ubuntu machines, the first step to do in my opinion is `create a root password` and continue all steps from here onwards with root user.
  1. *create root password*: `sudo passwd root`
  2. *change to root user*: `su -`

4. Configure `passwordless SSH` on both machines. This is crucial for both your Master and Slave nodes to be able to talk to each other. You might want to look up this [tutorial](https://www.google.com/url?hl=en-GB&q=https://medium.com/@luck/setup-passwordless-ssh-on-ubuntu-16-04-7ac81592fee6&source=gmail&ust=1510572674575000&usg=AFQjCNEU5Linm9tvZID-7wMjruzECz7YbQ) for this step.

  Apply the following steps on **both** machines:
  1. *install open-ssh server*: `apt-get install openssh-server`
  2. *generate ssh key*: `ssh-keygen -t rsa -P ""`
  3. *create an authorized_keys file and copy the key into it*: `cat ./.ssh/id_rsa.pub >> ./.ssh/authorized_keys`
  4. *give read and write permissions to the authorized_keys file*: `chmod 600 ~/.ssh/authorized_keys`
  5. On both the machine add the IP addresses of both the machines in the hosts file. The hosts file can be accessed using any text editor, example with nano: `nano /etc/hosts`. I added the following lines at the beginning of the file:
    - `MASTER_IP master`
    - `SLAVE_IP slave`
  5. *Enbale root access for SSH*: edit `/etc/ssh/sshd_config` file and change `PermitRootLogin` from **without password** to **yes**.
  6. *Restart SSH service on both machines*: `service ssh restart`

  After setting up your *ssh key* on your both your machines. Let's enable passwordless ssh to and from Master and Slave nodes. Perform the following on your respective nodes:

  **On Master**: *root@Master* ssh-copy-id -i root@Slave

  **On Slave**: *root@Slave* ssh-copy-id -i root@Master

  Verify passwordless SSH by `ssh root@Slave`

5. Install Scala on Ubuntu: `sudo apt-get update`. You can also refer to this [Github Gist](https://gist.github.com/mbonaci/eb0a1981628553bdb846)
6. Download and keep the relevant hadoop zip directory to provide its path in the next step.
7. Install and configure Apache Spark on Ubuntu. Perform the following steps on both the machines:

  1. Download relevant version of Apache Spark from [Spark Downloads](https://spark.apache.org/downloads.html) .
  2. Chose the relevant spark and hadoop versions and download the tar file from the given link.
  3. Move the zip file at place of your choice. I extracted it in root. Example path of Spark: */root/spark-2.2.0-bin-hadoop2.7*
  4. *Extract zip file*: `tar -zxvf spark-2.2.0-bin-hadoop2.7`
  5. Add the following lines to `.bashrc` files on both the machines:

    ```
    export SCALA_HOME=SCALA_PATH
    export SPARK_HOME=SPARK_PATH
    export PATH=$SCALA_HOME/bin:$PATH
    export HADOOP_HOME=HADOOP_PATH
    export PATH=$HADOOP_HOME/bin:$PATH
    ```
  6. Add the following lines to `.bashrc` files on both the machines to configure **pyspark**:
    ```
    function snotebook (){
      export PYSPARK_DRIVER_PYTHON="jupyter"
      export PYSPARK_DRIVER_PYTHON_OPTS="notebook"

    #if you use Python 3+ add this line else leave it
      #export PYSPARK_PYTHON=python3

    $SPARK_PATH/bin/pyspark --master local[2]}
    ```
8. Configure Spark using files in `SPARK_HOME/conf` directory on both the machines:

  1. Create copies of the following template files:
    - cp spark-defaults.conf.template spark-defaults.conf
    - cp spark-env.sh.template spark-env.sh
    - cp slaves.template slaves
  2. Add the following lines to the respective files:
    - *spark-defaults.conf*:
    ```
    spark.master spark://master:7077
    ```
    - *slaves*:
    ```
    master
    slave
    ```
    - *spark-env.sh*:
    ```
    export SCALA_HOME=/root/spark-2.2.0-bin-hadoop2.7
    export SPARK_WORKER_MEMORY=1g
    export SPARK_WORKER_INSTANCES=2
    export SPARK_WORKER_DIR=/root/workers
    export SPARK_MASTER_HOST=master
    ```
9. Fire up the Spark master: `SPARK_HOME/sbin/start-master.sh`

10. Fire up Spark slaves: `SPARK_HOME/sbin/start-slave.sh spark://master:7077` on each worker (master and slave) to start the workers

I hope this tutorial has provided you with a comfortable setup process for the given use case. I shall try to improve the guide from time to time.

Please feel free to create [Issues](https://github.com/daas-ankur-shukla/Guides-from-a-Novice/issues) on this repositories if you have any doubts regarding the contents of this blog or you find any errors or mistakes or write me at **work.ankurshukla@gmail.com**
