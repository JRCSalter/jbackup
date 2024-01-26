# JBACKUP

## Description

This is my personal backup script. I don't expect anyone to actually use it, because it's specific to my needs. But I'm putting it out there in case anyone finds some use out of it.

## My Setup

As this is personal to my needs, a little explanation of how my system is set up may be necessary.

I have a dual boot computer that primarily runs Linux, with Windows. Both OSes are stored on separate M.2 drives, but any files are saved to other drives. Apps are retained on the M.2 drives for speed.

I have a main working drive I have named `03-main`. This is the only drive I write to on a regular basis aside from the shared drives. The two shared drives are used only for Windows, and any files from them are copied over to `03-main` during this process. Then, any important files are copied from `03-main` to `mass_1-backup` (so named because it's a rather large drive with larger files on it that I am not backing up because I will need many more drives and the files aren't all that important). Files are also copied to Dropbox via a separate drive. All files are encrypted before syncing with Dropbox, and to avoid any permissions issues, I just set them to all.

Dropbox is on a separate drive just in case anything happens to Dropbox that causes files to get removed or corrupted that trickles down to my drive.

## Dependencies
* dropbox
* ecryptfs-utils
* rsync

## ecrypt-fs explanation

First, we check that dropbox is connected:

`mountpoint -q /mnt/dropbox`

Then we mount it with `ecryptfs`

`mount -t ecryptfs`

This is password protected, so we need to supply a password. Yes, this means that a password is stored in plain text somewhere on the computer. Not always good practice, I suppose, but your ssh keys are stored in a similar way, so I don't see too much of a problem here.

`-o key=passphrase:passphrase_passwd_file=$password_file`

The following just sets what kind of encryption you need

`-o ecryptfs_enable_filename_crypto=n -o ecryptfs_cipher=aes -o ecryptfs_key_bytes=32 -o ecryptfs_passthrough`

Finally I set the drive to be encrypted Still not entirely sure why it needs to be done twice, but it works.

`/home/$USER/Dropbox/encrypted /home/$USER/Dropbox/encrypted`

## rsync explanation

`rsync` allows me to copy files in a more precise way than `cp`.

### Options Used

* `-v` - Verbose. Not strictly needed in an automatic script, but useful if you manually run it.
* `-r` - Recursive, so we can run through all directories.
* `-u` - Update. Skip files that are newer on the receiver.
* `-l` - Copy symlinks as symlinks. It could be dangerous if all links are followed. This means links won't necessarily work, but if this is set up properly, then the relevant files are still copied.
* `-o` - Owner. Preserves the owner of the file.
* `-p` - Preserve permissions.
* `-g` - Preserve the group.
* `-t` - Preserve modification times.
* `--partial` - Keep partially transferred files. Useful if the script is aborted while copying larger files.
* `--progress` - Show the progress during transfer. Again, only really useful if running the script manually.
* `--delete` - Delete any files from the destination drive that have been deleted from the sending drive.

We can also set the permissions manually. Which is useful for Dropbox, which can get picky about permissions.

`rsync -vrulopgt --partial --progress --delete \`
  
  `--chmod=777 \`
  
  `--chown=jrcsalter:jrcsalter \`
  
  `/mnt/03-main/documents \`
  
  `/mnt/dropbox/Dropbox/encrypted`
