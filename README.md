<img width="1560" height="720" alt="kuva" src="https://github.com/user-attachments/assets/9adc0e38-cad7-4102-8f09-0e16c690ec9e" />

_In finnish - Suomenkielinen versio osoitteessa [iMac-toisena-monitorina-Linux-macOS-Windows
](https://github.com/Lumipyry/iMac-2009-2014-toisena-monitorina-Linux-macOS-Windows)_

Applicable with iMac models 2009-2014.

External display is set On and Off by pressing Powerbutton.

The main machine can have either macOS, Linux or Windows.

This is a step-by-step instruction for [smc_util](https://github.com/floe/smc_util) including [Powerbutton support](https://github.com/floe/smc_util/pulls) by [FreekMank](https://github.com/floe/smc_util/pull/11/commits). Needs to have Apple High Sierra or earlier (Dual boot with Linux) in the iMac which is used as external display.

Proven to work with MiniDisplay Port cable. No personal experiences with Thunderbolt cable. Tested with iMac models 2009-2011.

If the main machine has macOS another option is to use native screen mirroring/external display - function by installing macOS Sequoia with [Open Core Legacy Patcher](https://github.com/dortania/Opencore-Legacy-Patcher). A good video for 
[installing](https://www.youtube.com/watch?v=in5-3EjKFqA). Personally I got mirroring to work in an 2011 21" iMac by installing Sequoia but not in an 2010 27" iMac.
***

**NOTE**: You may want to leave `rc.local` out of the installation (`Step 16`) - if and when you want also use the Linux OS of the display machine. (`rc.local` makes the machine boot directly to Target Display Mode)

Without `rc.local` the machine boots first to Linux OS and pressing Powerbutton sets TDM on.

***

1.Download smc_util to home directory (`in the intended display iMac`)
```
git clone https://github.com/floe/smc_util.git
```
2.Move to smc_util
```
cd smc_util
```
3.Compile SmcDumpkey with GCC
```
gcc -O2 -o SmcDumpKey SmcDumpKey.c -Wall
```
4.Create files tdm_toggle.sh, powerbutton, powerbutton.sh and rc.local
```
touch tdm_toggle.sh powerbutton powerbutton.sh rc.local
```
5.Change content of file tdm_off.sh
```
#!/bin/bash
rm -f tdm_started
./SmcDumpKey MVHR 0
sleep 1
./SmcDumpKey MVMR 2
sleep 2
DISPLAY=:0.0 xrandr --output eDP --auto
```
6.Change content of file tdm_on.sh
```
#!/bin/bash
./SmcDumpKey MVHR 1
sleep 1
./SmcDumpKey MVMR 2
sleep 2
DISPLAY=:0.0 xrandr --output eDP --off
touch tdm_started
```
7.Create content to file tdm_toggle.sh
```
#!/bin/bash
pushd $(dirname "${BASH_SOURCE[0]}")

if [[ -f "tdm_started" ]]; then
  echo "Switching off TDM"
  source tdm_off.sh
else
  echo "Switch on TDM"
  source tdm_on.sh
fi

popd
```
8.Create content to file powerbutton
```
event=button/power PBTN
action=/etc/acpi/powerbutton.sh
```
9.Create content to file powerbutton.sh. `REMEMBER TO CHANGE XXXXXXXXX to your username (in TWO LINES in the script)`
```
#!/bin/bash

pushd $(dirname "${BASH_SOURCE[0]}")

FILE=powerbutton_pressed
NOW=$(date +%s)

if [[ -f "$FILE" ]]; then
  # Read timestamp of previous powerbutton press from file
  echo "File exists"
  typeset -i PREV=$(cat $FILE)
  echo Compare $NOW and $PREV

  # if two powerbutton presses were <1 seconds apart -> shutdown
  if [[ $(($NOW-$PREV)) -lt 2 ]]; then
    # Shutdown
    echo "Powerbutton pressed twice in a row: Shutting down"
    shutdown now
  else
    echo "Toggle TDM"
    /home/XXXXXXXXX/smc_util/tdm_toggle.sh &
  fi
else
  echo "Toggle TDM"
  /home/XXXXXXXXX/smc_util/tdm_toggle.sh &
fi

echo $NOW > $FILE

popd
```
10.Create content to file rc.local. `REMEMBER to change XXXXXXXXX to your username`
```
#!/bin/bash

# Start Target Display Mode such that the pc is used as external monitor right away
# Note: The Power button toggles TDM (see /etc/acpi/events)
pushd /home/XXXXXXXXX/smc_util
./tdm_on.sh
popd
```
11.Remove the SMC kernel driver to avoid conflicts
```
sudo rmmod applesmc
```
12.Make tdm_toggle.sh, rc.local and powerbutton.sh executable (`change XXXXXXXX`)
```
sudo chmod +x /home/XXXXXXXXX/smc_util/tdm_toggle.sh /home/XXXXXXXXX/smc_util/rc.local /home/XXXXXXXXX/smc_util/powerbutton.sh
```
13.Install build-essential and acpid
```
sudo apt install build-essential acpid acpid-suppport
```
14. Copy file powerbutton to /etc/acpi/events (`change XXXXXXXXXX`)
```
sudo cp /home/XXXXXXXXX/smc_util/powerbutton /etc/acpi/events
```
15.Copy file powerbutton.sh to /etc/acpi (`change XXXXXXXX`)
```
sudo cp /home/XXXXXXXXX/smc_util/powerbutton.sh /etc/acpi
```
16.Copy file rc.local to /etc (`change XXXXXXXX`)
```
sudo cp /home/XXXXXXXXX/smc_util/rc.local /etc/rc.local
```
17.Restart acpid
```
sudo systemctl restart acpid
```
18.Change the Powerbutton behaviour in Linux OS to
```
Do Nothing
```
