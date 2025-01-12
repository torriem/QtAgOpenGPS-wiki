## This only applies to adding properties to QtAOG. QtAgIO is a manual process

### Adding your own property:

Add property to parse_properties.py:
```

    { 'ini_path': 'seed/blockageIsOn',
      'cpp_name': 'property_setSeed_blockageIsOn',
      'cpp_default' : 'false',
      'cpp_type' : 'bool',
      'qml_name' : 'setSeed_blockageIsOn',
      'qml_type': 'bool',
      'qml_default': 'false',
    },

```

From QtAOG root, run:
``` 
./parse_properties.py /path/to/agopengps
```
Ignore warnings like <br>

"Warning! No ini path found for setVehicle_goalPointAcquireFactor. Generate props.py and fix."

## Update Properties to AOG Standards:

From QtAOG root, run: 
```
./parse_properties.py /path/to/agopengps
```
If you get a  warning like: <br>
"Warning! No ini path found for setVehicle_goalPointAcquireFactor. Generate props.py and fix."
Run 
```
python3 parse_properties.py /path/to/AgOpenGPS/SourceCode/GPS/Properties/Settings.settings /path/to/AgOpenGPS/SourceCode/GPS/Classes/CSettings.cs -d > newprops.py
```
Open newprops.py. <br>
Go to the line that has no value after the ":" (It'll be towards the end, and will look like this: 
```'setVehicle_goalPointAcquireFactor': '',```<br> <br>
Add ```vehicle/goalPointAcquireFactor``` in the ```''``` space, <br>
So it looks like ```'setVehicle_goalPointAcquireFactor': 'vehicle/goalPointAcquireFactor'``` <br>
Delete props.py <br>
Rename newprops.py to props.py <br> <br>
From QtAOG root, run <br>
```./parse_properties.py /path/to/agopengps``` <br><br>
Don't forget to commit the result!

