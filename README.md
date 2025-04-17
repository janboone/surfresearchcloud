# Additional configuration of SURF Research Cloud Ubuntu workspace
A standard Ubuntu workspace on SURF Research Cloud (SRC) has a very limited root filesystem (20-40GB in size). Because of this, leaving the server configured as it is by default can lead to the root filesystem running out of space due to increasing home directories, installation of applications, et cetera. To counteract this problem, you can follow the steps outlined in this document.

## Requirements
These instructions assume that you have sudo access on the workspace. Without this, you will not be able to perform the steps outlined in the instructions.

## Moving home directories
home directories residing on the root filesystem are a likely cause of running of diskspace on the root filesystem. In order to prevent this, the home directories can be moved to other storage.
To do so, start by creating extra storage in SURF Research Cloud by following the [instructions](https://servicedesk.surf.nl/wiki/spaces/WIKI/pages/19825226/External+storage+volumes) on the SURF Research Cloud Wiki. Make sure to give it a clear and recognizable name, such as _storage\_home_. We'll be using this name in the rest of the instructions.

## Technical support
The steps below can, if not properly executed, lead to an unreachable workspace on SRC. If you are a researcher at Tilburg University and you do not feel comfortable performing these steps yourself, please reach out to Digital Research Support at Tilburg University (digitalresearchsupport@tilburguniversity.edu and one of our engineers can walk you through the process).

### Pre-flight checks
After the new storage is available and connected to your workspace, go through the following steps to move the home directories to this new storage:
1. Log in to the workspace through SSH.
2. Run the command `df -h` to show the filesystems available within the workspace. Here you should see the extra storage you created earlier mounted as _/data/storage\_home_

If you don't see the new storage connected to your workspace, please check if you've followed the instructions on the SRC Wiki.

### Setting up root SSH access
As we'll be moving home directories which contain the SSH keys needed for users to log in, there is a risk that you won't be able to log back in to the workspace if something goes wrong. By default, the SRC Ubuntu workspaces are configured to disallow root SSH access[^1]. However, in order to be able to log into the workspace as root if something does go wrong with moving the home directories, we'll be making a temporary change to allow root SSH access.

**Warning:** Some of these steps will be performed as _root_. This is the most powerful account on a Linux machine, so please be careful when using this account.

Follow these instructions to enable root SSH access to the workspace:
1. Retrieve your public SSH key. You can see the key being used on the workspace with the following command: `cat ~/.ssh/authorized_keys`. Copy the public key, we'll be needing it later on. It will look something like this: _ssh-rsa a-whole-lot-of-characters your-username@host_. For the following steps, you need the public key up until the _your-username@host_ section, that last part is just a comment.
2. Switch to root with the command `sudo -i`.
3. Move the original _authorized\_keys_ file for root to _old\_authorized\_keys_ for safekeeping: `mv /root/.ssh/authorized_keys /root/.ssh/old_authorized_keys`
4. Add your own public key to a new _authorized\_keys_ file in order to be able to log into SSH as root: `echo '<YOUR OWN PUBLIC KEY HERE>' >> /root/.ssh/authorized_keys`
5. Set permissions on the new file: `chmod 600 /root/.ssh/authorized_keys`
6. Set ownership on the new file: `chown root:root /root/.ssh/authorized_keys`
7. Return to your normal user: `exit`

Now test if root SSH access is succesful by logging out of the workspace and log back in using _root_ as the username. If all is well, you'll now be able to log in as the _root_-user. If not, please make sure you've followed the steps above correctly. Please do not continue with the following steps if root SSH access is not correctly set up.

### Moving the home directories themselves
After you've confirmed that you're able to log into the workspace as root, it's time to move the existing home directories from the root filesystem to the newly attached storage.
If you've connected SURF Research Drive (SRD) to your workspace, we'll need to unmount SRD first:
1. First, check if SRD is mounted in any of the existing home directories: `mount | grep 'researchdrive'`. If you see any output from this command, SRD is mounted in at least one user's homedirectory. In order to unmount all mounted SRD, run the following command: `mount | awk '/researchdrive/ {print $3}' | while read path; do sudo umount "$path"; done`. Check if all SRD-mounts are now unmounted by repeating the command `mount | grep 'researchdrive'`. If all SRD are now properly unmounted, that command should not show any output.
2. Now copy over the existing home directories to the newly attached storage: `sudo cp -a /home/. /data/storage_home/`. Depending on the size of the home directories, this can take a little while. Please don't interrupt this process.
3. Then, update the _/etc/fstab_ file to mount the newly attached storage as _/home_:
    - Open _/etc/fstab_: `sudo nano /etc/fstab`
    - Look for the line that contains the path _/data/storage\_home_. This is the line you'll need to alter. By default, it'll look something like this: _UUID=<sequence of characters/numbers and dashes> /data/storage_home xfs defaults,nofail,x-mount.mkdir=777 0 0_
    - Change this line to look like this: _UUID=<sequence of characters/numbers and dashes> /home xfs nofail 0 2_. You'll be changing the path _/data/storage_home_ to _/home_, removing the _defaults_ and _x-mount.mkdir=777_ part, and changing the last _0_ to _2_.
    - Save the file with `Ctrl+O` followed by `Enter`. If this gives an error, use `Ctrl+X` followed by `n` to exit the file without saving and reopen the file. Make sure you've added `sudo` to the start of the command to open the file in the editor, altering the _/etc/fstab_ file requires root permissions.
    - Exit the editor with `Ctrl+X`
    - Check that your changes have been saved to the file with `cat /etc/fstab`. If the changes are not there, please carefully refollow the previous steps.
4. In this step, we'll be moving the existing homedirectory the newly attached storage as backup. This is not strictly necessary, removing the existing /home directory as root is also a valid approach. To move the existing home directories to the newly attached storage, execute the following commands: `cd /` (to make sure the logged in user isn't residing within _/home_ during the move), and then `sudo mv /home /data/storage_home/`. Please wait for this process to complete.
5. Whether you've removed or moved the _/home_ directory, execute the following commands to create a new fresh _/home_ directory ready to be mounted:
    - `sudo mkdir -m 755 /home`
6. Everything should now be set up correctly. Reboot the workspace with `sudo reboot` and try logging in as a regular user. If you've performed all the steps above correctly, you should be able to log in as a regular user without any issues and all home directories should now be moved from the root filesystem to the newly attached storage.
7. After logging back in, verify the new storage is mounted correctly by examining the `df -h` output again. You should now see the new storage mounted as _/home_ and no longer as _/data/storage_home_

### Restoring limitations to root SSH access
In the section _Setting up root SSH access_, we made some alterations to allow for root SSH access. If after rebooting you're able to log in as a regular user, please restore this to its original configuration with the command `sudo mv /root/.ssh/old_authorized_keys /root/.ssh/authorized_keys`. After this, trying to log in using SSH as root should again display the warning message and break the connection after 10 seconds.

### Running into technical difficulties
If you are a researcher at Tilburg University and you run into technical difficulties following these steps, please reach out to Digital Research Support at digitalresearchsupport@tilburguniversity.edu for assistance.

[^1]: Technically, SSH root access is not disallowed, but the workspace is set up to display a message warning the user not to use root for SSH access and then exit. This is configured in _/root/.ssh/authorized\_keys_ which we'll be altering in the instructions.
