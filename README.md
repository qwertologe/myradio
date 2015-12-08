myradio(1) --  An alarm clock with a simple audio player and weather forecast
=============================================================================

## SYNOPSIS

`myradio` [path_to_configfile.ini]

## DESCRIPTION

A highly configurable alarm clock with support for playing audio files from your filesystem and weather forecast from forecast.io. All works without X - framebuffer is enough.

Currently it works well with a raspberry and the new released [7 inch touch display](https://www.raspberrypi.org/products/raspberry-pi-touch-display/) - [more information](https://www.raspberrypi.org/blog/the-eagerly-awaited-raspberry-pi-display/).
Backlight control works with some flaws - see [KNOWN ISSUES].

You can configure which KV-screens you want to see [CONFIGURATION]. It may be possible, that a screen references another - e.g. 'player' has a link to 'alarm' (alarm settings).

The player works file-based - one reason, why i developed `myradio`. Support of filetypes depends on gstreamer and his plugins (the second reason).

Until now there are five screens:

  * Player: Main screen with audio player, digital clock, complete alarm view
  * Clock: Analog clock with remaining sleep time
  * Alarm: Alarm settings (two alarms)
  * Weather: Hourly weather forecast for today and tomorrow - powered by [forecast.io](http://forecast.io) - if you get an [apikey](https://developer.forecast.io)
  * System: The possibility to reboot/shutdown and update 'myradio' (using git)

## SCREENSHOTS

If you read this using `man`: Screenshots are not converted by `ronn`. Go to [GitHub](https://github.com/qwertologe/myradio) instead.

![alarm](https://cloud.githubusercontent.com/assets/1454849/10955131/13dcc236-8352-11e5-9976-8c9c5434af04.png)
![alarm_snooze](https://cloud.githubusercontent.com/assets/1454849/10955133/13e3f5f6-8352-11e5-843c-4abab8c360a7.png)
![clock](https://cloud.githubusercontent.com/assets/1454849/10955132/13df5e60-8352-11e5-89c5-0c68edb7966c.png)
![player](https://cloud.githubusercontent.com/assets/1454849/10956405/ae4fc8dc-835b-11e5-9179-93cc747c1bd3.png)
![weather](https://cloud.githubusercontent.com/assets/1454849/10955134/13e5a216-8352-11e5-8816-df07778d143c.png)
![system](https://cloud.githubusercontent.com/assets/1454849/10956058/a18e0436-8358-11e5-8803-d4565a9edba2.png)

## PREREQUISITE

It depends on your configuration. Here is, what i have done for my Raspberry PI:

  * [Install Python and Kivy](https://github.com/mrichardson23/rpi-kivy-screen) - maybe all steps upto 15 at the time of writing this document
  * Install additional required Python packages  `sudo pip install python-forecastio pytz tzlocal`
  * git must be installed (online update)

Weather (for details see [CONFIGURATION])

  * There is no problem if you have no APIKEY but you can't get weather forecast :-)
  * Get your APIKEY
  * Identify your language
  * Determine latitude and longitude

If your commands in `myradio.ini` use `sudo`, you must configure it.

As last part you need a `myradio.ini`:

## CONFIGURATION

It runs without an initial `myradio.ini` if you use the stable branch of kivy. Otherwise you must configure at least `monofont = RobotoMono-Regular` in section `[system]`. The alarm settings are updated in this file, too. Here follows a description of all configuration settings:

    vi myradio.ini # you see the default values with comments here

    [alarm]
    # path from which the analog clock will randomly pick an alarm file
    analog_dir = analog
    # same for digital clock
    digital_dir = digital
    # in how many seconds to alarm again after you have used "snooze"
    snooze_seconds = 60

    [alarm1]
    # alarm settings for alarm1
    # ...

    [alarm2]
    # ...

    [weather]
    # Determine latitude and longitude http://itouchmap.com/latlong.html
    latitude = 52.370235
    longitude = 4.903549
    # Get your APIKEY http://developer.forecast.io/
    apikey =
    # Identify your language https://developer.forecast.io/docs/v2#options
    language = en

    [audio]
    # comma-separated list of root directories for your audio files
    # use absolute paths - a path with / at the beginning
    # environment variables like $HOME get expanded to their values.
    dirs = $HOME
    # comma-separated list of extensions which will be shown in the player
    # and will be used for alarm, too
    # it's up to you, to configure your system for these formats (gstreamer)
    formats = flac,mp3,ogg

    [system]
    # for kivy >1.9 stable you can use RobotoMono-Regular
    monofont = DroidSansMono
    # comma-separated list of screen which will be shown
    # you can deactivate and order the screens here
    screens = player,clock,weather,alarm,system
    # For faster reboot/poweroff consider adding parameter "-f"
    reboot = sudo /sbin/reboot &
    shutdown = sudo /sbin/halt &
    # post_save_cmd will enable a button at the alarm settings
    # if you press the button, this command will be executed e.g.:
    # sudo mount -o remount,rw /;cp /run/shm/myradio.ini ~; sudo mount -o remount,ro /
    post_save_cmd =
    # backlight_cmd will be called if you change the brightness
    # a simple alternative if your display can be set with xbacklight:
    # backlight_cmd = xbacklight -set {}
    backlight_cmd = sudo sh -c "echo $(({}*255/100)) >/sys/class/backlight/rpi_backlight/brightness"
    # number of seconds to wait until the display is turned off (brightness 0)
    # touching the screen will revert the brightness (if possible)
    backlight_timeout = 120

    [git]
    # should return true if update is available, otherwise false
    # an empty check disables update button and check
    check_update_cmd = remote=`git ls-remote $(git rev-parse --abbrev-ref @{u} | tr / " ")` || false && test $(git rev-parse HEAD) != $(echo "$remote" | cut -f1)
    # pre could be used to remount rw if you use a ro filesystem
    pre_update_cmd =
    # pre could be used to remount ro if you use a ro filesystem
    post_update_cmd =
    update_cmd = git merge origin/master

    # Short version:

    [alarm]
    analog_dir = analog
    digital_dir = digital
    snooze_seconds = 60

    [weather]
    latitude = 52.370235
    longitude = 4.903549
    apikey =
    language = en

    [audio]
    dirs = $HOME
    formats = flac,mp3,ogg

    [system]
    monofont = DroidSansMono
    screens = player,clock,weather,alarm,system
    reboot = sudo /sbin/reboot &
    shutdown = sudo /sbin/halt &
    post_save_cmd =
    backlight_cmd = sudo sh -c "echo $(({}*255/100)) >/sys/class/backlight/rpi_backlight/brightness"
    backlight_timeout = 120

    [git]
    check_update_cmd = remote=`git ls-remote $(git rev-parse --abbrev-ref @{u} | tr / " ")` || false && test $(git rev-parse HEAD) != $(echo "$remote" | cut -f1)
    pre_update_cmd =
    post_update_cmd =
    update_cmd = git merge origin/master

## RECOMMENDATIONS

  * Use a network only runlevel or even better: Do not install an X server. It may be possible with [Minibian](https://minibianpi.wordpress.com/) in the future (Jessie will be released soon) or [Raspbian network install](https://github.com/debian-pi/raspbian-ua-netinst). This will reduce boot time and the time for software upgrades and at last the writes on your SD-card.
  * Configure your raspberry for read-only operation. This helps if you have temporary power outage and eliminates problems with your SD-card. Use tmpfs for `myradio.ini`, configure `post_save_cmd` and consider parameter '-f' for the shutdown/reboot commands.
  * At least use `noatime` option for your filesystems
  * Have a look at the [prerequisites](#PREREQUISITE) and setup.sh as a starting point
  * If you use a distribution with systemd and have problems mounting a NFS volume timely - here is a solution:

    192.168.178.1:/nfsexport /mnt     nfs     ro,noatime,nolock,noauto,x-systemd.automount  0      0

## KNOWN ISSUES

  * Backlight control is implemented but can only be on or off [forum for official 7-inch touch display](https://www.raspberrypi.org/forums/viewtopic.php?f=108&t=120968). Until now `/sys/class/backlight/rpi_backlight/brightness` can be set to a value from 0 to 255 but a value lower than 128 will turn it off, higher will turn it on. Ensure that the slider is above 50% which will result in a value >= 128. Additional, your touch to activate the brightness can affect a widget (if you hit one).
  * General code cleanup necessary (remove odds and ends from showcase)
  * Code quality is probably not the best - it is my first python and kivy project
  * No unit tests
  * Setting alarms in the next minute is not possible (comparable to cron)
  * Problems arise if you scroll the listview with touch. You can instead scroll it using the slider at the bottom [kivy issue 3418](https://github.com/kivy/kivy/issues/3418)

## WISH LIST 1

  * Error messages for post_save_cmd
  * Online update (git) for `myradio`

## WISH LIST 2

The following are not realizeable for me. Feel free to participate!

  * Maybe volume control, timer in the action bar
  * Icons: hail.png sleet.png wind.png (maybe needed in the future)
  * Themes (change color, iconset) on the fly
  * Nice graphics from an artist :-)
  * Additional KV screens - mine are all finished

KV/Kivy improvements

  * Improve KV layout for better performance (i use it on a Raspberry PI)
  * Device independence to make it useable for other displays
  * Fixes for the Listview problems (doubletab and scrolling with fingers is not useable)

## WISH LIST 3

I am not sure if there is really a need for it - but i am interested...

  * Generic settings KV(!) file for all configuration options with on-screen-keyboard - would need to move the remaining configuration variables to properties

## EXAMPLES

    myradio # normal start with myradio.ini in working dir
    myradio /run/shm/myradio.ini # start with specific ini

## AUTHOR

Michael Arlt

## LICENSE

    myRadio - an alarm clock with a simple audio player and weather forecast
    Copyright (C) 2015  Michael Arlt, GPL3 or higher - see LICENSE

Used source from Kivy Showcase: [MIT](https://opensource.org/licenses/MIT)

Most icons from gnome-accessibility-themes (HighContrast): presumably [GPL 2 or higher](http://www.gnu.org/licenses/old-licenses/gpl-2.0)

Logo icon based on:

  2010-07-20 Black windup alarm clock face by Sun Ladder - Own work.
  Licensed under [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) via Wikimedia Commons
  https://commons.wikimedia.org/wiki/File:2010-07-20_Black_windup_alarm_clock_face.jpg

Analog alarms:

  * [1.ogg](https://commons.wikimedia.org/wiki/File:Alarmclock-mechanical.ogg), Licensed under [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) via Wikimedia Commons
  * [2.ogg](https://commons.wikimedia.org/wiki/File:WWS_Alarmclock.ogg), Licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by/4.0/deed.en) via Wikimedia Commons
  * [3.ogg](https://commons.wikimedia.org/wiki/File:WWS_Alarmclockringing.ogg), Licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by/4.0/deed.en) via Wikimedia Commons

Digital alarms:

  * [1.ogg](https://commons.wikimedia.org/wiki/File:WWS_Alarmclockringing.ogg) (first release), Licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by/4.0/deed.en) via Wikimedia Commons

## THANKS

Thanks to the whole community - especially the task force from #kivy: bionoid, dessant, inclement, kovak, kived, tshirtman and all who i have forgotten.

## SEE ALSO

[kivy.org](http://kivy.org/)

