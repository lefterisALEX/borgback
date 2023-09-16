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
## dowload backup 

```
curl -Lo backup https://raw.githubusercontent.com/lefterisALEX/borgback/main/backup  
chmod +x backup    
sudo mv backup /usr/local/bin/
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

## Restore  a system (as root for the smart devices)  
1. Run the first steps to  install tha packages
2. generate ssh keypair
3. export variables
```
export box_user=$(aws --region=eu-west-1 ssm get-parameter --name "/homelabs/borg/user" --with-decryption --output text --query Parameter.Value)
export BORG_PASSPHRASE=$(aws --region=eu-west-1 ssm get-parameter --name "/homelabs/borg/passphrase" --with-decryption --output text --query Parameter.Value)
export box_url="ssh://${box_user}@${box_user}.your-storagebox.de:23/."
export ssh_key="~/.ssh/id_rsa.pub"
cat  /root/.ssh/id_rsa.pub | ssh -p23 ${box_user}@${box_user}.your-storagebox.de install-ssh-key
```
4. Modify the backup file and set the paths
example:
```
export box_remote_path="backup/homelabs/zigbee"
export local_path_to_backup="/home/pi/IOTstack"
export path_to_exclude="/home/pi/IOTstack/volumes"
```
6. export the remote path as variable and check the backups
```
export box_remote_path="backup/homelabs/zigbee"
borg list ${box_url}/${box_remote_path}

```
7. Restore file
```
cd /   # for IOTStack go to / , so will restore everything to /home/pi/IOTStack
borg list ${box_url}/${box_remote_path}
borg  extract ${box_url}/${box_remote_path}::2023-09-14_04:00
```
