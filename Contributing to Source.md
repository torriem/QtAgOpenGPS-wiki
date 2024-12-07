# Contributing to source

## Porting from AOG

If anything here is wrong, or something should be added, fork the repo and fix!
### Data types

The data types are slightly different between languages. Some, like double, int, are fairly universal.
But some are completely different.
- First type is C#, second is Qt/C++
- byte = quint8
- char = QByteArray
- string = QString
- Byte[] = QByteArray

### Other Useful Porting Notes
Random functions that I can never remember what the correct conversion is. <br>
C#: ```Buffer.BlockCopy(BitConverter.GetBytes(longitudeSend), 0, nmeaPGN, 5, 8);```
C++: ```std::memcpy(nmeaPGN.data() + 5, &longitudeSend, 8);```

## Creating scalable UI's

- This is very important. Creating a GUI that works on a phone screen as well as a giant desktop isn't easy. I thought I'd write this up because I don't want to convert your code to something that looks ok on a phone (and because it's more fun writing this than converting all my existing code to scalable like I need to).
 ### Every width, length, anchor margin that is coded to a number MUST have the scale factor calculated in. 
 - My first 6 months of code was written like:

    - ```width: 500```
    - ```height: 100```
    - ```anchors.left: 10```
    - ```anchors.top: 10```
    - etc etc

- Now if all you're going to do is use your application on is your computer, no issue. BUT try that on a phone and you'll regret ever trying to code. 
- The correct way is(as I learned yesterday):
    - ```width: 500 * theme.scaleWidth```
    - ```height: 100 * theme.scaleHeight```
    - ```anchors.left: 10 * theme.scaleWidth```
    - ```anchors.top: 10 * theme.scaleHeight```

- "theme.scaleWidth/scaleHeight" change based on screen size, so the GUI will look the same on a phone, just smaller.
- Now if your ```width: parent.width / 2```(or whatever), no scale factor is necessary, since parent will scale correctly.
- But if you go ```width: parent.width / 2 + 10```, you must change to ```width: parent.width / 2 + (10 * theme.scaleWidth)``` to figure in the scale factor.

- Obviously the width related sizes need ```theme.scaleWidth``` and the height related sizes need ```theme.scaleHeight```

### ```anchors.margins``` is off limits!
- If you go ```anchors.margins: 10```, then what do you set your scale factor to?? Is it a width setting, or a height?  
The answer is neither. So don't use it. Instead, go:
    - ```anchors.left: 10 * theme.scaleWidth```
    - ```anchors.top: 10 * theme.scaleHeight```
    - ```anchors.right: 10 * theme.scaleWidth```
    - ```anchors.bottom: 10 * theme.scaleHeight```
<br>  
    
<br>  
Easy, right? If you have questions, PLEASE ask someone, instead of writing 3000 lines of code with no scaling!
