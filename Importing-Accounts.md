When switching to a new code binary, you may need to **import your accounts** from the old binary.
In the following, we will describe the process of exporting accounts from a previous build and importing them to the latest release.

For the purpose of this tutorial we denote:
- `aion-old` the folder with the release that contains the accounts we want to import, and
- `aion-new` the new release that will be used in the future;

First, in a terminal, navigate to the old folder and get a list of all your accounts by typing:
```
cd aion-old
./aion.sh -a list
```
As output, you will get the list of public keys for your accounts, for example:
```
0xa0abcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdab
0xa012341234123412341234123412341234123412341234123412341234123412
```

Next, you need to obtain the private key associated with each account you want to export, by executing the command:
```
./aion.sh -a export 0xa0abcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdab
```
You will be prompted to input the password for that account, and then your private key will be displayed:
```
Please enter your password: 
Your private key is: 0xabcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234
```

You will use this private key to import the account into your new installation:
```
cd aion-new
./aion.sh -a import 0xabcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234abcd1234
```

After introducing the correct password associated with the account twice, the account will be imported:
```
Please enter a password: 
Please re-enter your password: 
The private key was imported, the address is: 0xa0abcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdab
```

You can verify that the account exists in the new installation by running:
```
./aion.sh -a list
```
which should now display the imported account:
```
0xabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcd
```