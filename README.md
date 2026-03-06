# How to sync Files between Linux and your Android phone via a drive service

For this tutorial we will use a linux PC and an android phone. In the end, we will be able to synchronize automatically files (in our case, an obsidian vault) between a PC and aSmartphone via multiple drive types.

**Requirements :** 

- Have a drive service compatible with rclone
- Have an android phone
- Have a linux PC
- Have a folder you want to start syncing on your PC (or phone, but we'll start from a pc in this tutorial.) Here, it will be : ~/Documents/Obsidian/Vault

# Tutorial

## Step 1 : Have a working drive.

- Install rclone on your pc
- So see if your drive is compatible with rclone, run `rclone config`, start setting up a new remote. If your drive shows in the list you should be good to go.
- In this tutorial, I will use google drive.

## Step 2 : Setup on PC side (linux)

- Set up a connection to your drive from your pc with the `rclone config` setup. You will be guided step by step by the program. We will name our drive "gdrive" but you can choose whatever you want.
- Once your drive is set up, try seeing the files in it with `rclone ls gdrive:`. If you see the contents of your drive listed, you're good to continue. Otherwise, there is an error during your setup and you will have to search a bit on your own.

## Step 3 : Syncing a local folder.
- Let's say we have a ~/Documents/Obsidian folder. We try to sync it with rclone.

Make your first sync with :

```shell
rclone bisync ~/Documents/Obsidian gdrive:yourremotepath/Obsidian/ --resilient --recover --max-lock 2m --conflict-resolve newer --resync
```

After that verify if all files have been correctly synced to your drive. If it's good, then you can setup a cronjob to run every x minutes automatically.

Open `crontab -e` and add this task :

```shell
*/10 * * * * rclone bisync ~/Documents/Obsidian/Vault/ gdrive:yourpath/Obsidian --resilient --recover --max-lock 2m --conflict-resolve newer
```

It's almost the same command as above but we wont use the --resync flag in it. This is setup to run every 10 minutes.

We are now done with the PC side. We only need to do it on the smartphone now.

> If crontab is not working, you will need to install cronie and enable it. On arch linux : `sudo pacman -S cronie && sudo systemctl enable crond`

## Step 4 : Setup things on the phone

- Install rclone and cronie on termux : `pkg install rclone cronie`
- Allow termux to access your android files with `termux-setup-storage`. This should open an android permission setup thing. Allow access to files. Android files can now be accessed from thje "storage" folder located at home.
- You can now choose where to sync your files from there. We will choose ~/storage/documents/Obsidian

Time to try to sync files on your smartphone :

Make your first sync with :

```shell
rclone bisync ~/storage/documents/Obsidian gdrive:yourremotepath/Obsidian/ --resilient --recover --max-lock 2m --conflict-resolve newer --resync
```

It will be a bit long depending on how many files you have to sync. 

If it all goes well, we will setup a cronjob on your termux to do it automatically from now on !

## Step 5 : Cronjob on termux

- We will do so that cronie starts everytime with termux. To do so, open your ~/.bashrc file and write this in it :

```shell
# Auto-start cron daemon if not already running
if ! pgrep -x crond > /dev/null; then
    crond
fi
```

- Now reload your shell : `source ~/.bashrc`
- See if cronie has started with `pgrep -l crond`. If you see a number like 12345, it worked.

- Now all you have to do is setup the task. Go `crontab -e` and paste this (dont forget to edit the paths according to your situation):

```shell
*/10 * * * * rclone bisync ~/storage/documents/Obsidian gdrive:yourremotepath/Obsidian/ --resilient --recover --max-lock 2m --conflict-resolve newer
```

## And voilà, you're set.

Of course you will maybe encounter issues depending on your situation, but this is the way I did it and it's working really well for me. I just wanted to share this, hope i helped !
