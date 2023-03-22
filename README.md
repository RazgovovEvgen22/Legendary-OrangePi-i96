# Legendary-OrangePi-i96
An image for the Orange Pi i96 made from fixes around the internet as well as a set of build system modifications (to the official OrangePi_Build tool) to be able to build it yourself if desired.  Debian Bullseye (11) is available.<br>
<br>
<strong>You do not have to build this yourself.</strong>  Images are available <a href="https://github.com/TheRemote/Legendary-OrangePi-i96/releases">here in the releases section</a>.<br>
<br>
My blog post that birthed this image is located <a href="https://jamesachambers.com/orange-pi-i96-getting-started-guide/">here on jamesachambers.com</a>.<br>
<br>
<h2>Fixes</h2>
<ol>
  <li>Adds fixed version of WiringPi tool as well as an altenative tool "opio" and fixes GPIO pin assignments</li>
  <li>Adds spidev interface to access SPI over the GPIO pins</li>
  <li>Adds OPi.GPIO library (able to drive SPI devices such as the Waveshare e-ink display)</li>
  <li>Adds ntp to assist with fixing time on first startup -- use "sudo ntpd -gq" to force a time sync once you've set your correct timezone (sudo dpkg-reconfigure tzdata)</li>
  <li>Adds ability to use the i96 as a HID device (mouse/keyboard emulation)</li>
  <li>Add CAN module support (MCP251x tested so far)</li>
  <li>Fixed SD card port to support "High Speed" timing mode (can be seen with sudo cat /sys/kernel/debug/mmc0/ios)</li>
  <li>Fixed Docker support (see Docker section below)</li>
  <li>Fixed TRIM support (sudo fstrim -av)</li>
  <li>Fixed notorious WiFi issues caused by missing crda package and no regulatory domain set (see first startup instructions to set REGDOMAIN)</li>
  <li>Fixed Bluetooth and set it up to work at startup using bluetooth patchram utility</li>
  <li>Fixed wireless and bluetooth MAC addresses changing each startup</li>
  <li>Fixed USB port to allow "High Speed" USB devices instead of locking them to "Full Speed"</li>
  <li>Fixed buggy UART not resetting properly which often breaks copying/pasting through a serial terminal</li>
  <li>Fixed sound issues that would prevent rebooting the system successfully after first startup</li>
  <li>Fixed GPIO permissions errors on startup</li>
  <li>Fixed low entropy (random numb er) issues by installing haveged service</li>
  <li>Fixed orangepi user missing group membership for audio, bluetooth, netdev, gpio</li>
</ol>

<h2>First Startup Instructions</h2>
Set correct timezone:
<pre>sudo dpkg-reconfigure tzdata</pre>
Sync time:
<pre>sudo ntpd -gq</pre>
Set correct locale:
<pre>sudo apt install locales -y && sudo dpkg-reconfigure locales</pre>
Set wireless regulatory country:
<pre>sudo nano /etc/default/crda</pre>
Add your two letter country code (mine is US) to the end of the bottom line (after the equals sign) that says REGDOMAIN=<br>
Press Ctrl+X then Y to save the file.

<h2>Docker Instructions</h2>
Docker will run on the image as of V1.36.  It requires the following change so that you don't get a service startup error:
<pre>sudo update-alternatives --set iptables /usr/bin/iptables-legacy</pre>
Now try starting the Docker service.  If it still won't start then clear everything out (warning: will clear all Docker containers and data so only do this if you are installing for the first time):
<pre>sudo rm -rf /var/lib/docker/*
sudo systemctl restart docker</pre>
This will remove all of Docker's existing cache and regenerate it.  It should start but it will be slow the first startup since it will rebuild the cache we just cleared.

<h2>Build Instructions</h2>
You should first clone the OrangePi_Build repository:<br>
<pre>git clone https://github.com/orangepi-xunlong/OrangePi_Build.git --depth=1
cd OrangePi_Build
./Build_OrangePi.sh</pre><br>
Choose the "Orange Pi i96" option which will create the "OrangePiRDA" folder.<br>
<br>
The files in this repository are meant to replace the stock OrangePiRDA ones that are generated (specifically in the scripts folder).  You simply copy them over the top of your generated folder like this:<br>
<pre>cd ..
git clone https://github.com/TheRemote/Legendary-OrangePi-i96.git
cp -R Legendary-OrangePi-i96/OrangePiRDA/* OrangePiRDA/</pre>
You may now build the image the same way I did!<br>
Ubuntu is not building correctly yet.<br>
<br>

<h2>Version History</h2>
<ol>
  <li>March 22nd 2023 - V1.37 - Update packages, add fixed version of serial adapter driver (thanks Nyanna, <a href="https://github.com/TheRemote/Legendary-OrangePi-i96/pull/6">PR #6</a>)</li>
  <li>October 24th 2022 - V1.36 - Fix USB audio kernel parameter, add more kernel parameters required by Docker</li>
  <li>October 23rd 2022 - V1.35 - Enable thin provisioning and bridge kernel modules</li>
  <li>October 14th 2022 - V1.34 - Enable USB audio devices and USB ethernet devices kernel modules</li>
  <li>October 13th 2022 - V1.33 - More spidev and CAN module improvements (thanks MZA)</li>
  <li>October 12th 2022 - V1.32 - Restore gpio_fixup service</li>
  <li>October 12th 2022 - V1.31 - Add USB Bluetooth driver support</li>
  <li>October 12th 2022 - V1.30 - Add CAN module support for native SPI without kernel rebuild (thanks MZA, <a href="https://github.com/MehdiZAABAR/OrangePi-I96-Work/">OrangePi-I96-Work</a>), Add libarchive-tools dependency to builder, remove gpio_fixup.sh as the board configuration has been corrected</li>
  <li>September 28th 2022 - V1.29 - Adjust lower bound of txpower curve to improve WiFi stability -- you can increase WiFi debugging verbosity with echo 4 > /sys/kernel/debug/rdawfmac/mmc*/dbglevel.  You can use 5 for the highest level of verbosity to help debug the driver (still lots of problems to find / solve and this will help see what the driver is doing)</li>
  <li>September 27th 2022 - V1.28 - Restore some WiFi regulatory settings such as country code '99' which has a special meaning in the cfg80211 driver</li>
  <li>September 27th 2022 - V1.27 - Fixed WiFi txpower driver settings (can be set between 10-20dBm with iwconfig wlan0 txpower 20 or iw dev wlan0 set txpower fixed 2000)</li>
  <li>September 27th 2022 - V1.26 - Add CAN module support for MCP251x (thanks MZA, <a href="https://github.com/MehdiZAABAR/OrangePi-I96-Work/">OrangePi-I96-Work</a>), add can-utils package</li>
  <li>September 26th 2022 - V1.25 - Removed deprecated default ssh options, more WiFi driver cleanup</li>
  <li>September 25th 2022 - V1.24 - Disable some WiFi crda / regulatory overrides hardcoded in drivers (there is now some regulatory activity in dmesg!).  There's more work to do on it but it's definitely better than it was</li>
  <li>September 25th 2022 - V1.23 - Fixed many dmesg errors related to WiFi, rda-mmc, and others</li>
  <li>September 25th 2022 - V1.22 - Ported wireless fixes to source instead of binary driver, more distributions.sh organization into functions, add packages wireless-tools/lshw/haveged/libnl-3-dev/libnl-genl-3-dev, use rootfs caching more efficiently for rebuilding</li>
  <li>September 24th 2022 - V1.21 - Fixed "unrecognized mount option 'hidepid=invisible' or missing value" error</li>
  <li>September 24th 2022 - V1.20 - Fixed SD card driver to correctly support high-speed mode, fixed TRIM support, decreased boot time by about 10 seconds by reducing rda-backlight timeout, cleaned up distributions.sh into organized functions</li>
  <li>September 23rd 2022 - V1.19 - Add OPi.GPIO library (supports devices such as the Waveshare e-paper display (thanks Michael, <a href="https://github.com/Farnsworth9qc/OPi.GPIO">OPi.GPIO fork</a>), add orangepi user to gpio group, general cleanup/removal of script portions related to other platforms than RDA</li>
  <li>September 22nd 2022 - V1.18 - Enable CONFIG_SND_USB to allow for USB soundcard use</li>
  <li>September 20th 2022 - V1.17 - Add further spidev fixes (thanks MZA, <a href="https://github.com/MehdiZAABAR/OrangePi-I96-Work/">OrangePi-I96-Work</a>)</li>
  <li>September 19th 2022 - V1.16 - Add cpufrequtils package (thanks Marco, <a href="https://github.com/TheRemote/Legendary-OrangePi-i96/pull/5">PR #5</a>)</li>
  <li>September 18th 2022 - V1.15 - Fix Bluetooth to have fixed MAC address stored in /data/misc/bluetooth (thanks Marco, <a href="https://github.com/TheRemote/Legendary-OrangePi-i96/pull/4">PR #4</a>)</li>
  <li>September 16th 2022 - V1.14 - Add bluez-tools package</li>
  <li>September 16th 2022 - V1.13 - Add Bluetooth patchram to enable bluetooth (thanks Marco, <a href="https://github.com/well0nez/RDA5991g_patchram">Bluetooth patchram fix</a>)</li>
  <li>September 14th 2022 - V1.12 - Add fixed WiringPi library (thanks MZA, <a href="https://github.com/MehdiZAABAR/WiringPi">WiringPi Fork</a>)</li>
  <li>September 10th 2022 - V1.11 - Fix USB gadget conflict (thanks SteveGotthardt, <a href="https://github.com/TheRemote/Legendary-OrangePi-i96/pull/3">PR #2</a>)</li>
  <li>September 10th 2022 - V1.10 - Adds USB serial connection. Set ACL to remove warnings from log files (thanks SteveGotthardt, <a href="https://github.com/TheRemote/Legendary-OrangePi-i96/pull/2">PR #2</a>)</li>
  <li>September 4th 2022 - V1.9 - Adds ability to use the i96 as a HID device (thanks jakeau, <a href="https://github.com/TheRemote/Legendary-OrangePi-i96/pull/1">PR #1</a>)</li>
  <li>September 3rd 2022 - V1.8 - Add crda package and instructions to configure regulatory domain (REGDOMAIN)</li>
  <li>September 2nd 2022 - V1.7 - Enable systemd-resolved service to help with DNS over WiFi, remove crashing hostapd service, fix e2scrub_all service, add orangepi user to audio, bluetooth, netdev</li>
  <li>September 2nd 2022 - V1.6 - Fix spidev</li>
  <li>September 1st 2022 - V1.5 - Adds spidev interface to access SPI over the GPIO pins</li>
  <li>August 31th 2022 - V1.4 - remove applying default locale due to breaking serial console on some systems (see first startup instructions to apply your correct locale instead of me applying mine which was causing problems), fix startup permissions errors related to GPIO</li>
  <li>August 30th 2022 - V1.3 - Add patb's gpio_fixup.sh script to fix GPIO pins on startup / gpio tool replacement / wireless LAN MAC address fix</li>
  <li>August 26th 2022 - V1.2 - Added ntp package to help with time issues (use "sudo ntpd -gq" on first startup), fixed locales issue</li>
  <li>August 24th 2022 - V1.1 - Added sound fix and UART fix from official OrangePi repository pull requests</li>
  <li>August 23rd 2022 - V1.0 - Initial Release</li>
</ol>
<h2>Credits</h2>
The drama surrounding this board is ridiculous.  Is it a great board?  Not really.  It also <a href="https://amzn.to/3QaYDeY">costs like $7 on Amazon</a>.  Is it a good board for $7?  Yes it is, especially in 2022 when Pis are routinely over $100.<br><br>
This board honestly isn't more broken than any other boards were at release.  It just never got support and it was almost right from the start.  It never received support from Armbian and Orange Pi has never supported it either (I mean most of the fixes in this were submitted to their official repository years ago with no response).  There are much more popular and widely used boards with much more serious hardware defects that are supported by both Armbian and Orange Pi (as they're well aware).<br><br> 
It's hard to say why everyone's jimmies were so rustled but from what I can tell it's mostly groupthink and political nonsense that has nothing to do with how defective the board actually is.  There were only 2 Pull Requests to even integrate from Orange Pi's repository as well as the USB issue.  Those are pretty much the only known issues with the board other than the flaky WiFI.  That is honestly a better track record than definitely most of Orange Pi's other boards and most boards in general.  Again, as someone who has been working with SBCs for nearly a decade and still has their Raspberry Pi 1 Model B's to prove it, what are you guys talking about?  This board is too toxic to touch with a flaky WiFI and some easily resolved kernel driver issues?  What?<br><br>
The lack of support is real though and has real consequences.  What hope is there then?  The community basically.  This image is just a compilation of fixes from people who dreamed of something better for the board and submitted PRs or fixes around the internet as well as patching Orange Pi's imaging tool to produce images that aren't literally 5 years outdated (although the kernel still is).<br><br>
Is it enough?  You'll have to judge for yourself if it's enough but it is enough to make this a decent headless Bullseye board with one working high speed USB 2.0 port.  The original ones really weren't which is why this was born.<br><br>
<br>
<a href="https://forum.armbian.com/topic/3232-orange-pi-2g-iot/page/6/">Credit to Gabor Hidvegi for the patch itself</a> as I found his patch (which wasn't as effective for the 2G version as he would have liked due to the modem initialization dropping the speed) to be working as-is for the regular i96 without the 2G modem present.<br>
<a href="https://github.com/orangepi-xunlong/OrangePiRDA_kernel/pull/2">Credit to GMMan</a> for the pull request on the official repository to fix sound playback kernel parameter issues<br>
<a href="https://github.com/orangepi-xunlong/OrangePiRDA_kernel/pull/3">Credit to GMMan again</a> for the pull request on the official repository to fix UART serial issues fixing copy/pasting<br>
<a href="https://wiki.pbeirne.com/patb/i96/src/master/gpio_fixup.sh">Credit to patb</a> for gpio_fixup.sh / devmem2.py which fixes the GPIO pins and the <a href="https://wiki.pbeirne.com/patb/opio">gpio tool replacement for WiringPi</a><br>
<a href="https://github.com/MesihK/linux-RDA8810/commit/d7089f4c43bd76082459e6995652b578ce2d10f4?diff=unified">Credit to MesihK</a> for the gpio files permissions fix<br>
<a href="https://4pda.to/forum/index.php?showtopic=813602&st=280">Credit to Yoshie</a> for enough hints to enable the spidev interface<br>
<a href="https://github.com/TheRemote/Legendary-OrangePi-i96/pull/1">Credit to jakeau</a> for adding the ability to use the i96 as a HID device (PR #1)<br>
<a href="https://github.com/TheRemote/Legendary-OrangePi-i96/pull/2">Credit to SteveGotthardt</a> for adding the USB serial connection and fixing ACL entries to eliminate warnings (PR #2)<br>
<a href="https://github.com/MehdiZAABAR/WiringPi">Credit to MZA</a> for fixing the WiringPi library to work with the i96 as well as fixes to the spidev interface<br>
<a href="https://github.com/well0nez/RDA5991g_patchram">Credit to Marco</a> for fixing the Bluetooth patchram utility to work with the i96 as well as fixing the Bluetooth MAC address (PR #4)<br>
<a href="https://github.com/Farnsworth9qc/OPi.GPIO">Credit to Michael</a> for adding i96 support to OPi.GPIO library.  Also see his <a href="https://github.com/Farnsworth9qc/e-Paper">Waveshare e-paper display fork</a> for getting a Waveshare e-ink screen working with the i96<br>
