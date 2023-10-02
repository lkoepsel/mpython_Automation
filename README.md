# MicroPython Automation using CoolTerm and Sublime Text

## Introduction
I created a *macOS* automated build for *MicroPython*, where I use the Build system in *Sublime Text* with the *Applescript* capability of *CoolTerm*. This enabled me to automate the following:
1. Disconnect *CoolTerm* from the serial port
1. Upload a file to the *MicroPython* board and reset the board
1. Reconnect *CoolTerm* and activate *CoolTerm* for interaction with the *MicroPython* board.

## Original Manual Process
I recommend the [three window programming setup](/posts/avr_c_edit/) as well. While the videos describes a *C* programming environment, the *MicroPython* setup would be the same, code editor, *CLI* and serial monitor. 

(**Note:** *A key feature of MicroPython, is that it automatically executes main.py upon boot. There are two programs which it will execute on boot, first, boot.py then main.py*)

1. Edit the file in *Sublime Text*
1. Switch to *CoolTerm* and disconnect it from the board (*CMD-k*) 
1. Switch to *CLI* to use *mpremote* to upload the file (*mpremote fs cp filename :main.py*)
1. Switch back to *CoolTerm* to reconnect (*CMD-k*) and view output
1. Press reset button on Pico to automatically execute *main.py*

## Required Programs

### mpremote
This automation requires the *MicroPython* utility, [*mpremote*](https://pypi.org/project/mpremote/) - Use pip to install *mpremote*. (*Which also means, you will need to have Python installed.*)

## CoolTerm
If you aren't already using *CoolTerm* then download [CoolTerm](https://freeware.the-meiers.org/) - multi-platform serial monitor **ONLY DOWNLOAD FROM THIS SITE:** https://freeware.the-meiers.org/
You will want to add two *CoolTerm* *Applescript* commands and allow the two scripts to interact with CoolTerm (do this via *System Preferences*)

**AND** after downloading *CoolTerm*, please take one more step to confirm the image is the proper one. Roger Meier has done a great job in providing the [checksums for the latest release](http://forums.the-meiers.org/viewtopic.php?p=2068&sid=2931b7c911099c94e20911aa455d511e#p2068). 

Please do the following and confirm your downloaded image checksum matches the proper one on the list in the forums:
1) Copy the appropriate hash based on your download from the [forums page](http://forums.the-meiers.org/viewtopic.php?p=2068&sid=2931b7c911099c94e20911aa455d511e#p2068).
2) Run the appropriate command to generate a hash on the downloaded program
3) Use the Python REPL to confirm they are identical (*examples below*)
### macOS and Linux example 
```bash
openssl sha256 /Users/lkoepsel/Downloads/CoolTermMac.dmg
SHA2-256(/Users/lkoepsel/Downloads/CoolTermMac.dmg)= 7972a2cc93d1a4ae25c5018f81c951d21930032d9ea51fa1e680ea53c6c860fb

# Confirm using Python REPL
# d = hash of downloaded file, r = hash from Roger's forums page, answer must be True
python3
Python 3.11.5 (main, Aug 24 2023, 15:09:45) [Clang 14.0.3 (clang-1403.0.22.14.1)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> r = '7972a2cc93d1a4ae25c5018f81c951d21930032d9ea51fa1e680ea53c6c860fb'
>>> d = '7972a2cc93d1a4ae25c5018f81c951d21930032d9ea51fa1e680ea53c6c860fb'
>>> r == d
True
>>>
```
**Note that the checksum generated matches exactly the checksum in the link!**

### Windows
For *Windows*, the command is different, however the process would be similar. The below example was run in Git Bash:

```bash
$ certutil -hashfile '/c/Users/Lief Koepsel/Downloads/CoolTermWin64Bit.zip' SHA256
SHA256 hash of C:/Users/Lief Koepsel/Downloads/CoolTermWin64Bit.zip:
70bd44f868834b14c3fbf18eced7b3563ca44f69d3422009681affb1a5c8c641
CertUtil: -hashfile command completed successfully.

# Confirm using Python REPL
# d = hash of downloaded file, r = hash from Roger's forums page, answer must be True
$ python -i
Python 3.8.10 (tags/v3.8.10:3d8993a, May  3 2021, 11:48:03) [MSC v.1928 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> d = "70bd44f868834b14c3fbf18eced7b3563ca44f69d3422009681affb1a5c8c641"
>>> r = "70bd44f868834b14c3fbf18eced7b3563ca44f69d3422009681affb1a5c8c641"
>>> r == d
True
>>>
```

Once you have confirmed the hashes are identical, install CoolTerm and enjoy!

### Sublime Text
For code editing use [Sublime Text](https://www.sublimetext.com), because it has this great build automation process.

## Simple Installation
The simple installation method is:
1. Copy the two script files to ~/bin/
1. Copy the *MicroPython.sublime-build* file to the User folder in Sublime Text->Settings->Browse Packages
1. In *Sublime Text*, be sure to check *Tools->Build System->MicroPython*. If this option doesn't appear after copying the file, restart *Sublime Text*
1. Use *Tools->Build With...* to show the three options, the first option explains the two variants, the first variant will copy the current file to *main.py* on the *MicroPython* board, the second will retain the existing name. Use the down arrow to select the variant then hit *Return*

## Installation with Explanations
### CoolTerm
The best method of creating these two files is:

1. Open ScriptEditor on your Mac
1. Copy and past the text below
1. Save the files with the titles as names, where ScriptEditor wishes to save them, typically iCloud/Script Editor folder
1. Copy them to a location where you keep local automation. For me, I have a folder called bin as in *~/bin* and I copy the two files there
#### CoolTerm_disconnect.scpt
```Applescript
# Disconnect CoolTerm from Board

tell application "CoolTerm"
	
	# Get the ID of the first open window
	set w to WindowID (0)
	if w < 0 then
		display alert "No open CoolTerm windows"
		return
	end if
	
	if IsConnected (w) then
		Disconnect (w)
	end if
	
	
end tell
```

#### CoolTerm_connect.scpt
```Applescript
use AppleScript version "2.4" -- Yosemite (10.10) or later
use scripting additions

# Connect CoolTerm to Board

tell application "CoolTerm"
	
	# Get the ID of the first open window
	set w to WindowID (0)
	if w < 0 then
		display alert "No open CoolTerm windows"
		return
	end if
	
	
	if Connect (w) then
		activate application "CoolTerm"
		return
		
	else
		# Attempt to reconnect twice before activating CoolTerm
		# This is to solve a race condition, where uPython doesn't always reconnect upon switching files
		display notification "Attempting to connect to CoolTerm." with title "uPython UploadScript"
		delay 1
		if (Connect (w)) then
			return
		else
			delay 1
			Connect (w)
		end if
		# after second attempt to connect, just activate CoolTerm and expect user to to connect
		activate application "CoolTerm"
	end if
	
	
end tell
```

### Sublime Text Build System
As *ST* is a programmer's editor, it has a [powerful automation process](https://forum.sublimetext.com/t/build-systems/14435/53) in which to create executable programs from edited files. It can be found under *Tools->Build System->* 

I added a new *Build System* which looks like this:
```bash
{
	"shell_cmd": "echo \" Use main to upload to main.py and same to retain filename\" ",

	"variants":
	[
		{
			"name": "CoolTerm main",
			"shell_cmd": "osascript ~/bin/CoolTerm_disconnect.scpt && mpremote cp $file :main.py && mpremote reset && osascript ~/bin/CoolTerm_connect.scpt"
		},
		{
			"name": "CoolTerm same",
			"shell_cmd": "osascript ~/bin/CoolTerm_disconnect.scpt && mpremote cp $file :$file_name && mpremote reset && osascript ~/bin/CoolTerm_connect.scpt"
		}

	]

}
```

To add the above as a *Build System*:
1. Go to *Tools->Build System->New Build System* 
1. Paste the *Build System* code above into the file which is opened by *ST* (replace the current contents of the file with the entire text above )
1. Save it with the name *MicroPython.sublime-build*
1. Check *Tools->Build System->MicroPython*
1. Click on *Tools->Build With...* pressing return will print a help message, or press the down arrow to the desired variant and press return.
1. Once the desired variant has run, a *CMD-b* will continue to run the same command.
