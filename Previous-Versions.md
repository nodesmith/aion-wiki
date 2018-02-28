If you are using aion-v0.1.8 or if you want a permanent id (used by peers), to connect to the Aion test network you need to first modify your configuration file to have a new personalized id.

Download the ID generation script generateId.sh [here](https://github.com/aionnetwork/aion/blob/1e5143711379623801ab59078d5af2d4e0cc9aa2/generateId.sh).
Add executable permissions to the script.

```
chmod +x generateId.sh
```

Run the script.

```
./generateId.sh
```

Copy the output.

Navigate to the `config.xml` file in `[aion_folder]/config/config.xml`:

```
cd config
gedit config.xml
```

Update the value between the id tags to the copied ID.

```
<id>my-new-id-value-is-set-here-12345678</id>
```

Versions aion-v0.1.9 and later do not require generating an id. A temporary unique id will be assigned to your kernel at runtime.

