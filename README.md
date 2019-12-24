# Install BloodHound on Ubuntu
Below is my experience of installing BloodHound on Ubuntu 18.04 Linux.

For the context, my system is a notebook with both Windows 10 and Ubuntu 18.04 installed. I installed both Operating Systems in a dual-boot configuration because I want to prevent any performance trade-offs due to hypervisors etc. 
*I initially tried with Ubuntu 19.10 but I failed to get that working so I re-tried with 18.04.*

I started with wiping out the (256GB SSD) drive completely and configuring UEFI SecureBoot in the notebook CMOS setup. Then created a GPT Windows 10 bootable USB stick with Rufus and installed Windows 10 on a 125GB partition*.
Then created GPT Ubuntu 18.04 bootable USB stick with Rufus and installed Ubuntu on another 125GB partition.

* Note that during Windows 10 installation 3 other partitions were automatically created, here's an overview of the disk layout after completing the installation:
/dev/sda1 555MB Microsoft Windows Recovery Environment (System - NTFS)
/dev/sda2 105MB EFI System (FAT32)
/dev/sda3 15MB Microsoft Reserved (Unknown)
/dev/sda4 125GB Basic Data (NTFS)
/dev/sda5 125GB Linux Filesystem (Ext4)

For the installation of neo4j and BloodHound I have used the guidance as described in an earlier post from vestjoe https://gist.github.com/vestjoe/68b579d07f6a685b15d05f55908883cc, however I needed to insert a 'sudo su' (see below) as without that it was throwing 'permission denied' messages to me.

...
sudo apt-get install wget curl git
wget -O - https://debian.neo4j.org/neotechnology.gpg.key | sudo apt-key add -
echo 'deb http://debian.neo4j.org/repo stable/' | sudo tee /etc/apt/sources.list.d/neo4j.list
echo "deb http://httpredir.debian.org/debian jessie-backports main" | sudo tee -a /etc/apt/sources.list.d/jessie-backports.list

sudo apt-get update
sudo apt-get install openjdk-8-jdk openjdk-8-jre
sudo apt-get install neo4j
...

sudo su
echo "dbms.active_database=graph.db" >> /etc/neo4j/neo4j.conf
echo "dbms.connector.http.address=0.0.0.0:7474" >> /etc/neo4j/neo4j.conf
echo "dbms.connector.bolt.address=0.0.0.0:7687" >> /etc/neo4j/neo4j.conf
echo "dbms.allow_format_migration=true" >> /etc/neo4j/neo4j.conf

The result of the above is that the following gets installed:
Configurations in
/etc/java-8-openjdk/
/etc/neo4j/neo4j.conf

Binaries in
/usr/share/applications/ (java)
/usr/share/neo4j/bin/

Logfiles in
/var/lib/neo4j/ (relevant files are neo4j.log and debug.log)

After installation, I had
$ java -version
openjdk version "1.8.0_232"
OpenJDK Runtime Environment (build 1.8.0_232-8u232-b09-0ubuntu1~18.04.1-b09)
OpenJDK 64-Bit Server VM (build 25.232-b09, mixed mode)

$ neo4j version
neo4j 3.5.14

While still logged on as su, continue with adding the BloodHound example database

git clone https://github.com/adaptivethreat/BloodHound.git
cd BloodHound
mkdir /var/lib/neo4j/data/databases/graph.db
cd BloodHound/
cp -R BloodHoundExampleDB.graphdb/* /var/lib/neo4j/data/databases/graph.db
neo4j start
netstat -lantp

Then log in to neo4j via the web interface http://localhost:7474/browser/
Initially both the username and password are neo4j, you must change the password (for example into BloodHound as long as you don't expose the database to the outside world)

The last step is to install the BloodHound binary application itself

On https://github.com/BloodHoundAD/BloodHound/releases select the correct binary for your Ubuntu system; BloodHound-linux-x64.zip
Now you have to decided where you want to place the software, I have placed it (extracted the ZIP archive) into /opt/BloodHound-linux-x64

Finally, you are able to launch the BloodHound application by selecting the directory containing the binaries and type ./BloodHound [Enter]
Then in the logon screen enter the database url bolt://localhost:7687, the username neo4j and the password (BloodHound).

Depending on your computer memory configuration you might want to make some additional adjustments in the /etc/neo4j/neo4j.conf file. Also take a look at the logfiles /var/lib/neo4j/neo4j.log and debug.log as these will likely give you additional hints on how to optimize the configuration.
