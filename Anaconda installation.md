# Installing Anaconda for all users
By default, Anaconda installs itself into the homedirectory of the user installing it. This can lead to bloated homedirectories. It can be preferable to install Anaconda in a location accessible to all users, so that environments can also be easily shared between users. This document outlines the steps needed to install Anaconda for all users on an Ubuntu workspace on SURF Research Cloud.

## Requirements
You'll need `sudo` access to your workspace in order to execute the steps in this document.

## Adding additional storage for Anaconda
If there isn't already additional storage attached to your SURF Research Cloud (SRC) workspace, follow the [instructions](https://servicedesk.surf.nl/wiki/spaces/WIKI/pages/19825226/External+storage+volumes) to add it.
After the storage is available and attached, log into your workspace and use the command `df -h` to see the mounted storage. There should be a line in the output with the following entry: _/data/name-of-your-storage_. For the rest of this document, we'll assume that the entry is _/data/anaconda_

## Downloading Anaconda
1. Download Anaconda as per the [instructions](https://www.anaconda.com/docs/getting-started/anaconda/install#linux-installer) on the Anaconda website.
2. Execute the command `bash ~/ANACONDA-INSTALLER.sh`, replace _ANACONDA-INSTALLER_ with the name of the file you've just downloaded.
3. Review and agree to the Terms of Service.
4. When prompted for the install location, do not accept the default install location, but enter _/data/anaconda_ in order for Anaconda to install to the newly attached storage.
5. Choose an initialization option depending on your preferences.

## Configure group and users
When the installation is complete, execute the following commands:
1. `sudo groupadd anacondausers`
2. `sudo chgrp -R anacondausers /data/anaconda`
3. `sudo chmod 770 -R /data/anaconda`
4. `sudo adduser <username> anacondausers`

Repeat step 4 for every user on the workspace you want to give access to Anaconda to.

## Configuring PATH for users
In order for users to find the Anaconda binaries, you'll need to automatically add them to all users's PATH. In order to do so:
1. `sudo nano /etc/profile.d/conda.sh`
2. Add the following line to this file: _export PATH=/data/anaconda/anaconda3/bin:$PATH_
3. Save the file with `Ctrl+O`, following by `Enter`
4. Exit the file with `Ctrl+X`
5. Check that the entry is correct in the file with `cat /etc/profile.d/conda.sh`
6. If the file looks good, set the file as executable with `sudo chmod +x /etc/profile.d/conda.sh`

Everything is now set up correctly. Log out and back in to have the new settings apply. All users added to the _anacondausers_ group should now have access to the Anaconda binaries. Also, any newly created environments will be accessible to all users in _conda env list_, assuming they've been created in the default location (_/data/anaconda/anaconda3/envs_)
