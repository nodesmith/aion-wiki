### Prerequisites

* [Ubuntu 16.04 or later version](http://releases.ubuntu.com/16.04/)
* [Oracle Java SE Development Kit 10](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
* [Apache Ant 1.10](http://ant.apache.org/bindownload.cgi)

### How to build

1. Clone the Aion project
```
git clone --recursive https://github.com/aionnetwork/aion 
```

2. Go into the folder 
```
cd aion
```

3. Check your environment settings are correct, including the Ant execute path, java execute PATH link to the JDK10 folder, and the JAVA_HOME. Then you should be able to build 
```
ant pack_build
or 
ant
```
4. Verify the code
You can go through the test cases by:
```
ant test
```

5. After build
check your **pack** folder
```
cd pack
```
You should see a file named like this
```
aion-v<KERNEL VERSION>.<GIT REVISION>-<BUILD DATE>.tar.bz2
```
eg. aion-v0.1.12.2c5119a-2018-02-28.tar.bz2

#### Build the fastVM and the solidity compiler native library from source code
If you want to build the project all from the source code. In step 3 of the How to build, you can use **ant full_build** to build all from source (It will take a while). 
Also, for the pre-reqiure environmet, you need:
1. Ubuntu 16.04 (if you want to build on top of the lastest ubuntu release, will require you to change the dependency of the build script in the aion_fastvm/script folder, the last ubuntu release doesn't has the libboost1.58 package)
2. linux kernel package **libboost1.58-all-dev**, **libjsoncpp1** and **cmake**


### Execute your Aion network
Please check the section **Aion Installation** in the [aion/README.md](https://github.com/aionnetwork/aion/blob/master/README.md)

### For Developers
To enable the google-java style pre-commit hook, open the command prompt and simply `cd` into the root of your Aion project and type:
`cp .gitsettings/pre-commit .git/hooks/`

Also note that to avoid style clashing with our license, at the top of each class that has a license displayed that begins with `/******...` (at least two * characters), simply add a single space between the first and second * characters as so: `/* *****...`
