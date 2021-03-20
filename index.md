<h3>Building an HDP Hadoop cluster</h3>
<p>Here are all the steps towards hadoop cluster.</p>
<p>Horton work data platform is a customized Hadoop developed by yahoo, the reason of choosing HDP is HDP offering API services that can be used in every third party applications 
</p>
<h4>Prerequisites:</h4>
<ul>
  <li>3 machines (virtual) (master, slave1, slave2) </li>
<li>Layer (Cloud services such as Azure, AWS, AliBaba Cloud or Oracle VM Box, Windows Hyper-V).</li>
<li>Our project will run under Oracle VM Box CentOS 7 img file (7+ GB, you can download it from CentOS official site).</li>
  </ul>

<h3>Preparation Steps:</h3>

<h4>Creating a 3 Machines</h4>
<ul>
<li>Click on “New” to create machine.</li>
<li>Name your machines (Master, Slave1, Slave2).</li>
<li>Select a path for each machine (better to setup them in an isolated partition (D:).</li>
<li>Centos is not available in the OS list, so you can select Redhat X64 instead.</li>
<li>Select the virtual machine RAM size (Minimum 4GB), HD size then finish.</li>
<li>Right Click on each machine and select “Settings”. Lets set the Network connection and the installation image:</li>

<li>Choose “Network” then set “Attached To: Bridget Adapter”</li>
<li>From “Settings” Also choose “Storage”</li>
<li>Under “Controller:IDE” Click on “Empty” then in “Optical Drive” choose the path that CentOS img is located inside.</li>
<li>VM Setup is done, now lets run the machines by click on “Start” The green Arrow in the top Bar.</li>
<li>After seconds, the setup will ask you to choose what you want to do with the machine, Of course we will install CentOS.</li>
<li>The Wizard is Easy (Select the OS Ver, Optical Drive, Location, Switch on the Network) then Hit Install/OK.</li>
<li>During Installation, the wizard will ask you to choose a password for the root user and to create a username, in our example the user is HdPUser and password is 123 and the root password is 123 also.</li>
</ul>
<p>Once instillation is finished, the system will ask you to do a reboot. The system will redirect you to the instillation steps again, to avoid this:</p>
<ul>
<li>From the top bar choose “Devices” >> “Optical Drive” >> “Remove Disk from Virtual Drive” the reboot “Machine” >> “Reset”.</li>
  <li>After the machine boot being complete, login with your username and password.</li>
  <p>"All of above are applied for all machines"</p>

<p>We need to go Root user</p>
<pre><code>
$ su root
</code></pre>
Hit your password
You are a Root user
Warning: To reach a successful Ambari installation, you have to log with the root user itself. (user: root; password:123)

<h4>Let update all machines</h4>
<pre><code>
$ yum update
</code></pre>

Warning: if the machine fail to contact the internet, run this command below and retry the update.
<pre><code>
$ dhclient (All Machines)
</code></pre>

<p>(Wait… "Drink some coffee till the steps got done")</p>

<h5>Putty is a must</h5>

<p>Due to the command copy/paste will be a master steps in this tutorial, we need to install putty. Thus, from putty official site, download it (x86 or x64) and install putty.
In order to use putty, we need the IP address for the machines that already setup.</p>
<pre><code>
$ yum install net-tools (All Machines)
</code></pre>
<p>To get the IP address for each machine</p>
<pre><code>
$ ifconfig (All Machines)
</code></pre>
<p>Now you have the ip address for all machine, just open Putty separately and paste the IP, then login to the machines as you did before a while, as a root.</p>

<h4>Setup Ambari pre-requisites.</h4>

<h5>Change the run level to multi user (all nodes)</h5>
<pre><code>
$ systemctl set-default multi-user.target 
$ systemctl get-default (to check if it multi user target)
</code></pre>

<h5>Change the host name (all nodes)</h5>
Warning: the names are master.hadoop.com for the master node, and slave1.hadoop.com for slave1 node, slave2.hadoop.com for slave2… 
<pre><code>
$ vi /etc/sysconfig/network
</code></pre>
press “i” to edit the file
new line
NETWORKING=yes
HOSTNAME=master.hadoop.com 
press Esc then :wq then Enter

<p>Apply this command (all nodes)</p>
<pre><code>
$ hostnamectl set-hostname master.hadoop.com (Dont forget to change the name for the other nodes)
$ hostnamectl status
</code></pre>

<h5>Disable firewall (All Nodes)</h5>

<p>Its important to disable firewall between nodes to avoid any chance of connection block.</p>
<pre><code>
$ systemctl stop firewalld 
$ systemctl disable firewalld 
$ systemctl status firewalld
</code></pre>
<p>It must be dead!</p>

<h5>Change VM Swappiness to 10 (All Nodes)</h5>

<p>This control is used to define how aggressive the kernel will swap
memory pages.</p>
<pre><code>
$ echo "vm.swappiness= 10" >> /etc/sysctl.conf
</code></pre>

<h5>Disable SE Linux (All nodes)</h5>
<pre><code>
$ vi /etc/selinux/config
</code></pre>
press “i”
set selinux from enforcing to disabled
press “Esc” then :wq to quit

<h5>ntp installing (All Nodes)</h5>
<pre><code>
$ yum -y install ntp 
$ systemctl enable ntpd 
$ systemctl start ntpd 
$ systemctl status ntpd (green active)
</code></pre>

<h5>Disable THP (All Nodes)</h5>
<pre><code>
vi /etc/rc.local
</code></pre>
<p>Delete this line First</p>
<pre><code>
touch /var/lock/subsys/local
</code></pre>
<p>and paste these lines and the end of the file</p>
<pre><code>
echo ‘Restarting network service’
service network restart
chmod +x /etc/aws114_bootlog.sh
/etc/aws114_bootlog.sh
touch /var/lock/subsys/local
$ echo “never” > /sys/kernel/mm/transparent_hugepage/enabled
$ echo “never” > /sys/kernel/mm/transparent_hugepage/defrag
</code></pre>

<h5>Update host file (All Nodes)</h5>
<pre><code>
$ vi /etc/hosts
</code></pre>
press i
start a new lines add this line ref for every node in your cluster
Example:
<pre><code>
192.1.1.1 master.hadoop.com master
192.2.2.2 slave1.hadoop.com slave1
192.3.3.3 slave2.hadoop.com slave2
</code></pre>
press Esc then :wq

Then paste this command "reboot". After reboot you will notice that the host name has been changed.
<pre><code>
$ reboot
</code></pre>
To make sure about your hosts name
<pre><code>
$ cat /etc/hosts
</code></pre>

<h4>Setup Ambari server (Master Node only)</h4>

<p>Download and install Ambari server in master node only on the browser navigate to this link:</p>
<p>https://docs.cloudera.com/HDPDocuments/Ambari-2.7.0.0/bk_ambari-installation/content/ambari_repositories.html</p>

Choose the ambari version on Centos7 “Repo file” if you want.

Back to master machine
<pre><code>
$ cd /etc/yum.repos.d 
$ yum -y install wget 
$ wget http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.7.0.0/ambari.repo 
$ yum -y install ambari-server ambari-agent
</code></pre>
<p>Do the same thing with other machines but install only ambari agent</p>
<pre><code>
$ cd /etc/yum.repos.d 
$ yum -y install wget 
$ wget http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.7.0.0/ambari.repo 
$ yum -y install ambari-agent
</code></pre>

<h4>Setup All ambari agents to connect to the Master node (All Nodes)</h4>
<p>Get the master Hostname and past it in all ambari agents (master node)</p>
<pre><code>
$ hostname -f
</code></pre>
<p>Get the hostname and go to all ambari agents (All nodes) only to paste the host name there.</p>
<pre><code>
$ vi /etc/ambari-agent/conf/ambari-agent.ini
</code></pre>
go to [server] information and change localhost to your master node name <master.hadoop.com>

<h5>Setup Ambari server (Master node only)</h5>
<pre><code>
$ ambari-server setup
</code></pre>
<p>Accept the default settings</p>
<pre><code>
$ ambari-server start
</code></pre>
<p>Start ambari-agent on all hosts</p>
<pre><code>
$ ambari-agent start (All nodes)
</code></pre>
<p>Add some permission to the user that running Ambari server already</p>
<pre><code>
$ chown -R /var/run/ambari-server (master node only) 
$ chown -R /var/run/ambari-agent (all nodes)
</code></pre>

<h4>Run Ambari server on Web browser</h4>

<h4>"MasterIpAdress":8080</h4>

<p>username and password is admin, admin</p>

<h5>Download and install Hadoop Echo system and run the cluster</h5>

<p>The steps now will run will the Ambari server wizard</p>
<ul>
<li>Start a new Cluster</li>
<li>Click On Launch install Wizard</li>
<li>Give your cluster a name then NEXT</li>
<li>Select your version then Click NEXT</li>
<li>Write your hosts full name including the master node (FQDM) as a list names. Ex:</li>
  <p>master.hadoop.com</p>
  <p>slave1.hadoop.com</p>   
  <p>slave2.hadoop.com</p>
<li>In the same page (Host registration information) choose Perform manual registration on hosts and do not use SSH, due to we setup Amabri agent already.</li>
<li>In the host registration, if you did everything right, your host registration will success in green. Click NEXT.</li>
<li>Select the services you want to install then click NEXT.                                                     
<p>Warning: to make it easy, just choose the minimal ecosystems that can start hadoop normally are: HDFS, MapReduce,Yarn and Hbase. note that the system will force you to choose some other services, just agree and select next.</p>                               
<p>Warning: do not choose the services that required a pre-installed database such as hive and oozie, in the next session i’ll show how to install them.</p>

<li>“Assign masters” page, change on which hosts you want to install the services then click NEXT.</li>
<li>“Assign Slaves and Clients” choose what to install in masters and nods then Click NEXT.</li>
<li>Please provide credentials for all services, i suggest you to use the same username and password for all services.</li>
<li>Last step is validation, then click Deploy to start the installation session.</li>
</ul>
  <p>50 Min is the required time for a success installation.</p>
<p>If the installation Fail, check the error and click "Retry</p>
<p>Done, your cluster is ready.</p>
