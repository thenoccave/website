# Docker WSL Integration missing resource

Recently I had to setup Docker desktop on my corporate laptop to do some debugging on a script that utilised Docker for all the wrong reasons.

When trying to set it up the WSL machine wasn't showing up under the WSL Integrated Resources. When debugging it also wasn't showing up when using wsl --list in order to upgrade to WSL v2.

## TL;DR
My day to day account isn't a local admin on my computer. I do have an account that is but use UAC or runas to escalate to it when required. The WSL machine runs as my non-admin account. When lauching Docker Desktop because it was complaining about needing to be a member of the Docker Users group I was running it with my admin account. Also when using the WSL command I was using an elevated powershell prompt.

This meant Docker Desktop and the WSL were running as my admin account and not showing the WSL server. Adding my corporate AD account to the local docker-users group on the machine and using wsl from a non-elevated command prompted allowed me to upgrade the WSL instance from v1 to v2. Then when running Docker from my corporate non-admin account the WSL machine was available in the WSL Integrated resource settings screen.

## The long version
When trying to get Docker working with an existing WSL machine on my corporate laptop. Using the WSL command to try upgrade from v1 to v2 which I assumed was the cause didn't show the machine either.
![wsl --list missing machine](/img/wsl-list-admin.png)
The biggest clue was this warning when starting Docker Desktop as a non-admin user
![Docker Desktop permission denied](/img/docker-desktop-docker-users.png)
Looking at the local groups there is a group called docker-users
![Local groups](/img/docker-desktop-computer-management.png)
Adding my AD account to this group allowed Docker Desktop to start running as my non-admin account
![docker-user group](/img/docker-desktop-add-user.png)
Running wsl --list as my local non-admin account showed the missing machine.
![wsl non-admin](/img/wsl-list-nonadmin.png)
Then upgrading to WSL v2 the machine now showed under the WSL Integrated resources in Docker Desktop.
![wsl docker resource](/img/docker-desktop-conversion.png)