# MicroPython Automation using CoolTerm and Sublime Text

## Introduction
I wanted to create a *macOS* automated build for *MicroPython*, where I could use the Build system in *Sublime Text* with the *Applescript* capability of *CoolTerm*. This enabled me to automate the following:
1. Disconnect *CoolTerm* from the serial port
1. Upload a file to the *MicroPython* board and reset the board
1. Reconnect *CoolTerm* and activate *CoolTerm* for interaction with the *MicroPython* board.

## Original Manual Process
I recommend the [three window programming setup](/posts/avr_c_edit/) as well. While the videos describes a *C* programming environment, the *MicroPython* setup would be the same, code editor, *CLI* and serial monitor. 

(**Note:** *A key feature of MicroPython, is that it automatically executes main.py upon boot. There are two programs which it will execute on boot, first, boot.py then main.py*)

1. Edit the file in *Sublime Text*
1. Switch to *CoolTerm* and disconnect it from the board (*CMD-k*) 
1. Switch *CLI* to use *mpremote* to upload the file (*mpremote fs cp filename :main.py*)
1. Switch back to *CoolTerm* to reconnect (*CMD-k*) and view output
1. Press reset button on Pico to automatically execute *main.py*

## Required Programs

### mpremote
This automation requires the *MicroPython* utility, [*mpremote*](https://pypi.org/project/mpremote/) - Use pip to install *mpremote*. (*Which also means, you will need to have Python installed.*)

## CoolTerm
If you aren't already using *CoolTerm* then download [CoolTerm](https://freeware.the-meiers.org/) - multi-platform serial monitor **ONLY DOWNLOAD FROM THIS SITE:** https://freeware.the-meiers.org/
You will want to add two *CoolTerm* *Applescript* commands and allow the two scripts to interact with CoolTerm (do this via *System Preferences*)

### Sublime Text
For code editing use [Sublime Text](https://www.sublimetext.com), because it has this great build automation process.

## Simple Installation
The simple installation method is:
1. Copy the two script files to ~/bin/
1. Copy the *MicroPython.sublime-build* file to the User folder in Sublime Text->Settings->Browse Packages
1. In *Sublime Text*, be sure to check *Tools->Build System->MicroPython*. If this option doesn't appear after copying the file, restart *Sublime Text*
1. Use *Tools->Build With...* to show the three options, the first option explains the two variants, the first one will copy the current file to *main.py* on the *MicroPython* board, the second will retain the existing name.

## Installation with Explanations
### CoolTerm
The best method of creating these two files is:

1. Open ScriptEditor on your Mac
1. Copy and past the text below
1. Save the files with the titles as names, where ScriptEditor wishes to save them, typically iCloud/Script Editor folder
1. Copy them to a location where you keep local automation. For me, I have a folder called bin as in *~/bin* and I copy the two files there
#### CoolTerm Disconnect
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

#### CoolTerm Connect
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
