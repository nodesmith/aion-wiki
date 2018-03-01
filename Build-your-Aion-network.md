### Prerequisites

* [Ubuntu 16.04 or later version](http://releases.ubuntu.com/16.04/)
* [Oracle Java SE Development Kit 9](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
* [Apache Ant 1.10](http://ant.apache.org/bindownload.cgi)

### How to build

1. Clone the Aion project
```
git clone https://github.com/aionnetwork/aion 
```

2. Go into the folder 
```
cd aion
```

3. Grab latest commits from server 
```
git submodule update --init --recursive
```

4. Above command will set current branch to detached HEAD. set back to master. 
```
git submodule foreach git checkout master
```

5. When you want to update the latest submodule codebase, otherwise the fresh build can ignore this step. 
```
git submodule foreach git pull origin master
```

6. Check your environment settings are correct, including the Ant execute path, java execute PATH link to the JDK9 folder,and the JAVA_HOME. Then you should be able to build 
```
ant pack_build
or 
ant
```
7. Verify the code
You can go through the test cases by:
```
ant test
```

8. After build
check your **pack** folder
```
cd pack
```
You should see a file named like this
```
aion-v<KERNEL VERSION>.<GIT REVISION>-<BUILD DATE>.tar.bz2
```
eg. aion-v0.1.12.2c5119a-2018-02-28.tar.bz2


### Execute your Aion network
Please check the section **Aion Installation** in the [aion/README.md](https://github.com/aionnetwork/aion/blob/master/README.md)


