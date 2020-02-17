#!/bin/sh
#
# Enable and Disable Palo Alto Networks Global Protect VPN software.
#
# This software can cause problems for some users because it keeps
# trying to connect, popping up dialog boxes, with no way to quit.
#
#   * The UI does not have any kind of "Quit" button or menu item.
#
#   * The macOS Activity Monitor can use "Quit" or "Force Quit",
#     and the command line process tools can use `kill` or `pkill`, 
#     but the software restarts when it's loaded into launchctl.
#
#   * The software is able to run simultaneously on multiple user
#     accounts on the same Mac; this is an atypical setup for a solo
#     user, but a typical setup for our teammates, who may have a user
#     account for normal work, and a separate user account for demos.
#
# The kill function uses `launchctl` to `unload the code,
# then kills all processes named GlobalProtect and PanGPS.
# 
# The start function reloads the code with its dependencies.
#
##
set -uf

# setup our usage text
usage="$(basename "$0") kills and starts the Palo Alto GlobalProtect processes to keep them
from always running in the background.
Commands in this script require sudo privileges and will request a password.

usage: $(basename "$0") [-k] [-s]
where:
    -h | --help         shows this helper text
    -k | --kill         unloads and kills all related processes (requires sudo)
    -s | --start        loads all processes for GlobalProtect connections"

# exit on interrupt
function on_sigint() { 
    exit 1; 
}

# launch main and helper processes for GlobalProtect
function startGlobalProtect()
{
    launchctl load -w /Library/LaunchAgents/com.paloaltonetworks.gp.pangps.plist >/dev/null 2>&1
    launchctl load -w /Library/LaunchAgents/com.paloaltonetworks.gp.pangpa.plist >/dev/null 2>&1
}

# kill all GP processes, some commands take a few seconds
function killGlobalProtect()
{
    # Sudo the echo command to force a password prompt right away
    # so the user isn't surprised later
    sudo echo Unloading services...
    launchctl unload -w /Library/LaunchAgents/com.paloaltonetworks.gp.pangpa.plist >/dev/null 2>&1
    launchctl unload -w /Library/LaunchAgents/com.paloaltonetworks.gp.pangps.plist >/dev/null 2>&1

    while true; do
    sudo killall GlobalProtect >/dev/null 2>&1 || break
    sleep 1
    done

    while true; do
    sudo killall -9 GlobalProtect >/dev/null 2>&1 || break
    sleep 1
    done

    while true; do
    sudo killall PanGPS >/dev/null 2>&1 || break
    sleep 1
    done

    while true; do
    sudo killall -9 PanGPS >/dev/null 2>&1 || break
    sleep 1
    done
}

# by default, launch Global Protect if no params are passed
if [ $# -eq 0 ]
then
        startGlobalProtect
        exit
fi

while [[ $# -gt 0 ]]
do
key="$1"

case $key in
    -k|--kill)
        killGlobalProtect
        exit
        ;;
    -s|--start)
        startGlobalProtect
        exit
        ;;
    -h|--help)
        echo "$usage"
        exit
        ;;
esac
done
