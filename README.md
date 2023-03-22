# borgback
A basic bash script to automate borgback of home labs to a Hetzner box.  
Modify the exported variables accordingly.  
I am using Paramater store for the passphrase and the user, but can be exported with other way too.

```
box_remote_path        :   The remote path in the Hetzner box. 
local_path_to_backup   :   The local directory you want to backup
path_to_exclude        :   Excude this drectory from the backups
box_user               :   The Hetzner box user
BORG_PASSPHRASE        :   Your secret passphrase.Yuu need this decrypt the backup when you restore.
ssh_key                :   Change this if you want to import another ssh public key to the box.
```

## install

```
sudo apt-get install info unzip borgbackup -y
curl https://awscli.amazonaws.com/awscli-exe-linux-${ARCH}.zip -o awscliv2.zip ; unzip awscliv2.zip ; sudo ./aws/install --update ; rm -rf awscliv2.zip
```

## initialize repository
Before initialize you can export some variable

```
export BORG_PASSPHRASE=<your passphrase>
export user=<your box user>
```

```
cat  /root/.ssh/id_rsa.pub | ssh -p23 ${user}@${user}.your-storagebox.de install-ssh-key
borg init  --encryption=repokey  --make-parent-dirs ssh://${user}@${user}.your-storagebox.de:23/./backup/homelabs/<server>/<app>
```

## Backup operations
```
borg list ssh://${user}@${user}.your-storagebox.de:23/./backup/homelabs/<server>/<app>

borg create -v --stats \
ssh://${user}@${user}.your-storagebox.de:23/./backup/homelabs/<server>/<app>::'{now:%Y-%m-%d_%H:%M}' \
/home/ubuntu/apps/wirehole  \
--exclude path_you_want_to_exclude

borg  ssh://${user}@${user}.your-storagebox.de:23/./backup/homelabs/<server>/<app>::<backup_name> 
borg mount ssh://${user}@${user}.your-storagebox.de:23/./backup/homelabs/<server>/<app>::<backup_name>  /<mount_point>
```
