# My build of dmenu v5.0

![](https://hostr.co/file/ftPxHjOAW14k/dmenu.png)

This repository hosts the source code of my build of dmenu (Dynamic Menu) from the [Suckless software](https://tools.suckless.org/dmenu/). It is based on dmenu v5.0 and different patches have been applied in order to provide the features I liked. The list applied patches can be found in the *patches* folder. It features the following settings:

* Case insensitive matching: search will match both capital/non-capital characters
* Adjustable line height
* Letters matching the search are highlighted with a color

The highlight has its own individual color as illustrated in the following screenshot:

![](https://hostr.co/file/oT6u6w9qvsWd/dmenu-fuzzy.png)

# Dependencies
The following packages are necessary in order to run this build of dmenu properly:

* ttf-jetbrains-mono
* ttf-joypixels

The fonts are hard-coded in the *config.def.h* file but can be changed if you don't want to use the two above.

# Installation
Basically, just clone this repository (or download it), compile the build and install it. Type the following commands:

```
git clone --depth 1 https://github.com/GSquad934/dmenu.git
cd dmenu
sudo make clean install
```

This build integrates nicely with [my custom build of DWM](https://github.com/GSquad934/dwm).

# Running dmenu
The simplest way to run dmenu is to type the following command:

```
dmenu_run
```

Obviously, several options for dmenu are available in order to modify the prompt shown and what the results will be. For example, the following command would simply display the prompt *Launch: * as showcased in the above screenshots:

```
dmenu_run -p "Launch: "
```

## Scripts

The real power of dmenu is revealed when using it with custom scripts. In a general way, the scripts are combined to a key binding so they can be quickly called.

The two sub-chapters below provide two examples of scripting with dmenu.

### Example #1: ACPI operations
In order to display a dmenu prompt for performing ACPI operations such as shutdown/reboot the computer or logout, the following code can be used:

```
#!/bin/bash
OPTS=$(echo -e "Shutdown\nReboot\nLogout" | dmenu -i -p "Choose: ")
case $OPTS in
        Shutdown) systemctl poweroff ;;
        Reboot) systemctl reboot ;;
        Logout) xdotool key "super+shift+q" ;;
        *) ;;
esac
```

When running this script, only three choices will be displayed: Shutdown, Reboot and Logout.

### Example #2: run an application as a GUI/TUI
By default, dmenu does not have the ability to determine if an application needs to be ran in a terminal or not. This script allows you to choose if a program needs to be run in a terminal or not + it builds a list of known applications so it will remember if programs are terminal based or not. Here is the code:

```
#!/bin/bash
TERM="alacritty -e"
CACHE=${XDG_CACHE_HOME:-$HOME/.cache}/dmenu-recent-apps
mkdir -p ${XDG_CACHE_HOME:-$HOME/.cache}
CONFIG=${XDG_CONFIG_HOME:-$HOME/.config}/dmenu
mkdir -p $CONFIG
touch $CONFIG/apps-term
touch $CONFIG/apps-bg

MOST_USED=`sort $CACHE 2>/dev/null | uniq -c | sort -rn | colrm 1 8`
RUN=`(echo "$MOST_USED"; dmenu_path | grep -vxF "$MOST_USED") | dmenu -p Execute: "$@"` || exit
(echo $RUN; head -n 99 $CACHE 2>/dev/null) > $CACHE.$$
mv $CACHE.$$ $CACHE

# figure out how to run the command based on the first word.  Note that this does not support
# a bg/term decision based on further arguments (although you could easily add that)
word0=${RUN%% *}
match="^$word0$"
while ! grep -q $match $CONFIG/apps-term $CONFIG/apps-bg
do
    type=$(echo -e "term\nbg" | dmenu -p type)
    [ $type = bg -o $type = term ] || continue
    echo $word0 >> $CONFIG/apps-$type
done
grep -q $match $CONFIG/apps-term && exec $TERM $RUN
grep -q $match $CONFIG/apps-bg && exec $RUN
```

# Contact
You can always reach out to me:

* [Twitter](https://twitter.com/gaetanict)
* [Email](mailto:gaetan@ictpourtous.com)