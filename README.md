encrypt-rclone
---

A script to archive, encrypt and rclone files to a remote host.

## Description

The motivation for this script is to make it easy to store backups on a cloud or other storage provider, and to prevent the provider from being able to examine or process the files being backed up. Each upload will create a new stand-alone encrypted archive with a timestamped name. The archives are encrypted using a one-off symmetric password, which must be entered again to decrypt the archive.

## Dependencies

1. tar
2. gpg
3. docker (rclone is run inside a Docker container)

## Install

1. Put the `encrypt-rclone` script somewhere, e.g. `/opt/encrypt-rclone`.
2. Set up rclone config in `~/.config/rclone/rclone.conf`.

You can use rclone to help auto-generate the config, by running:
```sh
docker run --rm -it -v ~/.config/rclone:/config/rclone rclone/rclone:latest config
```

Note: if your config is stored in a different location, ensure that you set the following env var when running the script:
```
RCLONE_CONFIG_DIRPATH=/path/to/my/rclone-config-dir
```

## How to use

### Put

1. Run the script manually, where `remote` is the name defined in the rclone config, and `remote-directory` is the directory to store the encrypted archive:
```sh
./encrypt-rclone put <remote>:<remote-directory> /path/to/my/dir1:/path/to/my/dir2
```
2. Enter password and press enter.
3. Log in to your provider and confirm that the encrypted archive has been uploaded as expected.

You may wish to automate the upload. You can set the password as an environment variable `ENCRYPT_RCLONE_PASSWORD`.

A cron job could be set up like this for example:
```sh
# Backup at 9am every day
echo "ENCRYPT_RCLONE_PASSWORD=mypassword" >> /etc/crontab
echo "0 9 * * * /opt/encrypt-rclone >> /var/log/encrypt-rclone.log" >> /etc/crontab
```

You may wish to use further scripts to ensure that the password is stored more securely than this.

### Extract

1. Download the backup from its storage in your preferred way.
2. Run the script, passing in the filepath:
```sh
./encrypt-rclone extract /path/to/my/backup-20191105-153940.gpg
```
3. Enter password and press enter.
4. The decrypted archive will be placed in the same directory.
5. Extract the archive:
```sh
tar -xvzf /path/to/my/decrypted_backup-20191105-153940.tar.gz
```

## Todo

make it fail properly if the upload doesn't work
expose other rclone options
make it work with an existing gpg key
