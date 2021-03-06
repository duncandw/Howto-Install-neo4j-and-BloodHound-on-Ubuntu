# Install BloodHound on Ubuntu
Below is my experience of installing BloodHound on Ubuntu 18.04 Linux.

For the context, my system is a notebook with both Windows 10 and Ubuntu 18.04 installed. I installed both Operating Systems in a dual-boot configuration because I want to prevent any performance trade-offs due to hypervisors etc.

*I initially tried with Ubuntu 19.10 but I failed to get that working so I tried again with 18.04 - not sure if that really makes the difference however.*

### Computer CMOS setup and disk partitioning

I started with wiping out the (256GB SSD) drive completely and configuring UEFI SecureBoot in the notebook CMOS setup. Then created a GPT Windows 10 bootable USB stick with https://rufus.ie/ and installed Windows 10 on a 125GB partition.
Then created GPT Ubuntu 18.04 bootable USB stick with Rufus and installed Ubuntu on another 125GB partition.

Note that during Windows 10 installation 3 other system partitions were automatically created, here's an overview of the full disk partition layout after completing the installation:
- /dev/sda1 555MB Microsoft Windows Recovery Environment (System - NTFS)
- /dev/sda2 105MB EFI System (FAT32)
- /dev/sda3 15MB Microsoft Reserved (Unknown)
- /dev/sda4 125GB Basic Data (NTFS)
- /dev/sda5 125GB Linux Filesystem (Ext4)

### Credits go here

For the installation of neo4j and BloodHound I have used the guidance as described in an earlier post from vestjoe https://gist.github.com/vestjoe/68b579d07f6a685b15d05f55908883cc, however I needed to insert a `sudo su` as without that it was throwing some 'permission denied' messages to me.

Anyway, let's start installing...

### Installing java and neo4j

```
sudo apt-get install wget curl git
wget -O - https://debian.neo4j.org/neotechnology.gpg.key | sudo apt-key add -
echo 'deb http://debian.neo4j.org/repo stable/' | sudo tee /etc/apt/sources.list.d/neo4j.list
echo "deb http://httpredir.debian.org/debian jessie-backports main" | sudo tee -a /etc/apt/sources.list.d/jessie-backports.list

sudo apt-get update
sudo apt-get install openjdk-8-jdk openjdk-8-jre
sudo apt-get install neo4j
```
Now here's the part where I added the `sudo su` in order to make things work;

```
sudo su
echo "dbms.active_database=graph.db" >> /etc/neo4j/neo4j.conf
echo "dbms.connector.http.address=0.0.0.0:7474" >> /etc/neo4j/neo4j.conf
echo "dbms.connector.bolt.address=0.0.0.0:7687" >> /etc/neo4j/neo4j.conf
echo "dbms.allow_format_migration=true" >> /etc/neo4j/neo4j.conf
```

### Verify installation results

The result of the above is that the following gets installed:
Configurations in:
```
/etc/java-8-openjdk/
/etc/neo4j/neo4j.conf
```
Binaries in:
```
/usr/share/applications/ (java)
/usr/share/neo4j/bin/
```
Logfiles in:
```
/var/lib/neo4j/
``` 
(relevant files are neo4j.log and debug.log)

After installation, I verified installing the above components with:

```
$ java -version
openjdk version "1.8.0_232"
OpenJDK Runtime Environment (build 1.8.0_232-8u232-b09-0ubuntu1~18.04.1-b09)
OpenJDK 64-Bit Server VM (build 25.232-b09, mixed mode)

$ neo4j version
neo4j 3.5.14
```

### Installing the BloodHound example database

While still logged on as `su`, continue with adding the BloodHound example database:

```
git clone https://github.com/adaptivethreat/BloodHound.git
cd BloodHound
mkdir /var/lib/neo4j/data/databases/graph.db
cd BloodHound/
cp -R BloodHoundExampleDB.graphdb/* /var/lib/neo4j/data/databases/graph.db
neo4j start
netstat -lantp
```

### Testing neo4j and changing the initial password

Then log in to neo4j via the web interface http://localhost:7474/browser/
Initially both the username and password are neo4j, you must change the password (for example into BloodHound as long as you don't expose the database to the outside world)

![neo4j via browser](https://github.com/duncandw/Install_BloodHound_on_Ubuntu/blob/master/neo4j_screen.png)

The last step is to install the BloodHound binary application itself

On https://github.com/BloodHoundAD/BloodHound/releases select the correct binary for your Ubuntu system; (BloodHound-linux-x64.zip). 
Now you have to decide where you want to place the software, I have placed it (extracted the ZIP archive) into /opt/BloodHound-linux-x64.

Finally, you are able to launch the BloodHound application by selecting the directory containing the binaries and type
```
./BloodHound
```

Then in the logon screen enter the database url bolt://localhost:7687, the username neo4j and the password (BloodHound).

![BloodHound logon](https://github.com/duncandw/Install_BloodHound_on_Ubuntu/blob/master/BloodHound_Logon.png)

Depending on your computer memory configuration you might want to make some additional adjustments in the /etc/neo4j/neo4j.conf file. Also take a look at the logfiles /var/lib/neo4j/neo4j.log and debug.log as these will likely give you additional hints on how to optimize the configuration.

### Using BloodHound in the field

When using BloodHound in the field, you will experience that it is slow, very slow. I initially had 8GB of RAM in my notebook, I bought a 16GB extension so now have 24GB in total but still not performing very well. Interestingly enough when running the Cypher queries directly in the neo4j browser it performs quite well, but then it of course is tough to actually understand the result of the graph.
One thing that I didn't fix yet is to extend the 'maximum open files', that might still help to improve performance. But I'm happy anyway that it actually works now :+1:.
