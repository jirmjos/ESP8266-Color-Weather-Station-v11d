##  Enhanced ESP8266 Color Weather Station -- (Development Edition v11d)(based on original by Daniel Eichorn (squix78) )

This project is a series of enhancements made to the original color weather station for the ESP8266 created by Daniel Eichorn, with contributions by several others.  Though the original display layout came from squix78, I've made substantial changes to add additional information.

This version of the project was written specifically for the WeMos Mini D1 with the 2.2" color LCD display based on the ILI9341 controller that squix78 used.

This edition (v11d) is a beta-release that is currently being debugged, but seems to be pretty stable.  Use at your own risk.

#  Overview of Changes from v8

This release contains significant changes related to the weather icons.  As I ran the original code from squix78 for several months, I began to notice that a few icons were not properly indicating either the current conditions or forecast.  First, the icon for "chancetstorms" did not exist, so no icon would be displayed for this condition.  I originally installed a temporary fix in the code and replace this condition with the one from "tstorms" which worked fine, but for a perfectionist like me, I had to find a permanent fix.  Also, as we got into winter weather, I noticed that I never saw an icon indicating snow, flurries, or chance of snow.  These all were indicated with icons for sleet.  And finally, one thing that continued to bother me was seeing the sun showing the current condition at night!  All of this led me to figuring out how I could add or modify my own icons.

After some digging, I think I located the original icon set used by squix78 and downloaded a local copy.  I then set about ensuring I had one for the chance of thunderstorms, as well as modifying those for nighttime conditions with clear-to-partly clear conditions to show a moon image instead of a sun.  I then dug through some of the code to find out why ambiguous icons were being displayed at times (e.g., sleet instead of snow), and found that the Weather Underground client library function for the weather icon translation returned ambiguous results for these.  As a fix, I bypassed this routine so as to not break any existing code, and added a new function that returned the actual icon text from the Weather Underground API call.  I modified my code to use this new function to retrieve the proper icon images.

Finally, I added some new logic to detect when it was nighttime, and use this to determine when to display the new nighttime icons as appropriate.  (Actually, I set a window of 30-minutes after sunset to 30-minutes before sunrise as nighttime to be precise.)

I also added a "splash screen" at startup with attributions to Weather Underground, the key developers, and creators of the original icon sets.

All of these bitmaps are arranged in multiple directories for better organization from the original.  In this release, the icons originate locally in the sketch "/data" directory, and are copied to the ESP8266 filesystem (SPIFFS) via the ESP8266 Sketch Data Upload tool (see below for instructions).  I did retain squix78's code that will look for the key icons at startup, and if not found, will still download from his site for now.  (These are the original icons though, not my changes!)

Many other enhancements were made in this release, including many bug-fixes.  Most were related to the display of weather alerts and long forecast texts that appear in the bottom panel.  I've changed the approach for the weather alert indicators so these appear as "reverse" images, or white text on a colored background.  This was my original intent, but the library does not support specifying background colors for custom fonts, so I had to roll my own routine for this.

One other noticeable display change was the addition of a WiFi signal-strength indicator.  I borrowed this idea from squix78's original weather station for the SSD1306 OLED display, but rewrote the bar-strength display routine to more closely mimic what is found on cell-phones and other mobile devices.  I'm also displaying the strength as a percentage above the bars.  Maybe overkill, but I had some issues as one point with a weak WiFi signal on mine and was getting missing / old data on the display.  The cause wasn't immediately obvious, so I added this indicator along with a number of other error checks and display changes.  The time will now show "--:--" if the NTP update fails after several attempts.  Also, several error checks were added that will result in retries if the periodic updates for weather data fail.  This was done to avoid a lengthy delay in updating some key data, such as astronomy info, that isn't normally updated very often.  If an update failed previously, the data wasn't updated until the next period, possibly leaving missing / old data on the display for a long time.

#  Quick Start

(*** Added 4/9/17 ***)
1.  Install the libraries that are required as noted in the --v8 release in this repository.
1.  Wire the WeMos (or equivalent) as noted.
1.  Install the ESP8266 Sketch Data Upload tool from here:  https://github.com/esp8266/arduino-esp8266fs-plugin
1.  Download and unzip this repository to your system.
1.  Ensure your board is set correctly with the appropriate flash size (4M, 3M SPIFFS) in the IDE.
1.  Compile, download, and run the SPIFFS format tool located in the "/ESP SPIFFS Format" folder.  This will give you a clean SPIFFS file system area.  Be patient...it takes a few minutes to format the filesystem.
1.  Compile and download the Weather Station v11d code to the WeMos.
1.  Run the ESP8266 Data Upload tool located in the Tools menu of the IDE.  You should receive information in the IDE indicating the copying of the icon folders located in the "/data" folder to the ESP8266.
1.  The initial configuration is done via a configuration portal or webpage. The first time you bring up the app, if it cannot find previously stored settings, it will bring up an access point and configuration portal. Look for an access point that begins with "ESP..." and connect to it. The access point is password protected, with a default password as "portal-pass".  It is a captive portal, and in most cases a browser will open automatically connecting to the portal after connection. If it doesn't, then type the address "192.168.4.1" in a browser to bring it up once connected to the AP. It should be fairly self-explanatory at that point; enter your WiFi AP/password credentials, WeatherUnderground API key, WU weather station, timezone city from the list, etc., and click "Save". The ESP should restart and try to connect using the new configuration. There's other features of the configuration portal, such as scanning for all available access points and allowing you to just select one, but again, it's pretty self-explanatory once you see the configuration page.  (If you've done this before with a prior release, you'll need to reconfigure and save the settings, as the SPIFFS format will erase the configuration file stored previously.
1.  If it doesn't connect, then you can at any time bring up the configuration portal again manually by double-clicking the reset button on the WeMos D1 mini. (This doesn't have to be done really fast, just two resets within 10 seconds.) You should also see information on the LCD display when it comes up in config mode.
1.  The initial defaults for the configuration portal are located on the "Settings.h" tab as well. If you look through this, you'll find parameters for the WU API key, the timezone city, weather station, etc. These are currently set for my location and preferences, but you can change them here if you want also.

