This repo contains the fix I've made for the plr.sh file of my Neptune 4, which uses software V1.1.2.41

I do not guarantee that this fix is safe, and works with your slicer.

All it should do, is to remove the starting comments (such as model image and printer parameters) from your gcode in a more strict way, compared to the stock script. I did not bother trying to fix the original line, but the end result should not be affected.

I've also added the "Does the file exist?" check, which prints its result into the console, and removed the "set fan speed to 80%" part, as it is utterly pointless.

* * *

In order to put this file onto your machine, you have to:
1. Download it, in, say C:/Example/modified_plr.txt
2. Download and install Putty and PSCP at https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
3. With printer connected to your local network, open the Windows PowerShell at the folder containing pscp.exe
4. (Optional, but recommended) Type pscp -r mks@IP_ADDRESS_OF_YOUR_PRINTER:/home/mks/plr.sh C:/Example/plr_recovery.txt and press submit, making a recovery file of your original plr.sh
5. Type pscp -r C:/Example/modified_plr.txt mks@IP_ADDRESS_OF_YOUR_PRINTER:/home/mks/plr.sh

You should now be able to press resume on the Power Loss Recovery window of your printer (if opened) and be good to go.
