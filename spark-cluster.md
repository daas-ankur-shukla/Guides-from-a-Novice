# Setting Up A Two Machine Spark Cluster
## Ubuntu Guest (VirtualBox) on a Windows host

Lately I took up a project which required a sample 2-machine spark cluster. Having two laptops at my hostel room, inspired me to set it up myself.
Setting up a cluster on Windows seemed unlikely, although I had found a blog on that too (Ref:). The path I chose was to set it up on Ubuntu installed as VirtualBox guests on two Windows machines.

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

4. Configure `password less SSH` on both machines. This is crucial for both your Master and Slave nodes to be able to talk to each other. You might want to look up this [tutorial](https://www.google.com/url?hl=en-GB&q=https://medium.com/@luck/setup-passwordless-ssh-on-ubuntu-16-04-7ac81592fee6&source=gmail&ust=1510572674575000&usg=AFQjCNEU5Linm9tvZID-7wMjruzECz7YbQ) for this step. Apply the following steps on **both** machines:
  1. *install open-ssh server*: `apt-get install openssh-server`
  2. *generate ssh key*: `ssh-keygen -t rsa -P ""`
  3. *create an authorized_keys file and copy the key into it*: `cat ./.ssh/id_rsa.pub >> ./.ssh/authorized_keys`
  4. *read and write permissions to the authorized_keys file*: `chmod 600 ~/.ssh/authorized_keys`
  5. 
