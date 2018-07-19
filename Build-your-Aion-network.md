### Prerequisites

* [Ubuntu 16.04 or later version](http://releases.ubuntu.com/16.04/)
* [Oracle Java SE Development Kit 10](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
* [Apache Ant 1.10](http://ant.apache.org/bindownload.cgi)
* 8GB RAM

### How to build

1. install pre-required package
``` 
sudo apt-get update && install -y git libboost1.58-all-dev libjsoncpp1 cmake g++ llvm-4.0
```

2. Clone the Aion project
```
git clone --recursive https://github.com/aionnetwork/aion 
```

3. Go into the folder 
```
cd aion
```

4. Check your environment settings are correct, including the Ant execute path, java executes PATH link to the JDK10 folder, and the JAVA_HOME. 

This is the example for the building environment settings in **.profile** under the home path. You can reload it
by **source ~/.profile**
```
export JAVA_TOOL_OPTIONS=-Dfile.encoding=UTF8
export JAVA_HOME=$HOME/IDE/java/jdk-10.0.1
export ANT_HOME=$HOME/IDE/apache-ant-1.10.3
PATH="$ANT_HOME/bin:$JAVA_HOME/bin:$PATH"
```

Then you should be able to build 
```
ant pack_build
or 
ant
```
6. Verify the code
You can go through the test cases by:
```
ant test
```

6. After build
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
If you want to build the project all from the source code. In step 4 of the How to build, you can use **ant full_build** to build all from source (It will take a while). 
make sure your Ubuntu is 16.04 release (if you want to build on top of the latest Ubuntu release, will require you to change the dependency of the build script in the aion_fastvm/script folder, the last Ubuntu release doesn't have the libboost1.58 package)


### Execute your Aion network
Please check the section **Aion Installation** in the [aion/README.md](https://github.com/aionnetwork/aion/blob/master/README.md)

### For Developers
To enable the google-java style pre-commit hook, open the command prompt and simply `cd` into the root of your Aion project and type:
`cp .gitsettings/pre-commit .git/hooks/`

Also note that to avoid style clashing with our license, at the top of each class that has a license displayed that begins with `/******...` (at least two * characters), simply add a single space between the first and second * characters as so: `/* *****...`
