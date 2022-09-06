# Install MySQL Cluster Manager "MCM" 8.0 using systemd

For more information on how to create a service look at ```man systemd.service```

### Install

Directory for MCM installation is /opt
Download latest generic tar package of MCM from edelivery.oracle.com or support.oracle.com
(today this is package: MySQL Cluster Manager 8.0.30 TAR for Generic Linux x86 (64bit))

Run commands below once you have downloaded the MCM package:
```
sudo su -
cd /opt
tar xzf /tmp/mcm-8.0.30-linux-glibc2.12-x86-64bit.tar.gz
mv mcm-8.0.30-linux-glibc2.12-x86-64bit mcm
cp /opt/mcm/mcm8.0.30/docs/sample_mcmd.conf /opt/mcm/mcmd.conf
mkdir /opt/mcm/mcm_data
```
Set data folder to folder /opt/mcm/mcm_data in configuration file:
```
cat /opt/mcm/mcmd.conf | sed 's/^#data_folder.*/data_folder = \/opt\/mcm\/mcm_data/' >> /opt/mcm/mcmd.conf.tmp
mv /opt/mcm/mcmd.conf.tmp /opt/mcm/mcmd.conf
```
Set logging folder to folder /opt/mcm in configuration file:
```
cat /opt/mcm/mcmd.conf | sed 's/^#logging_folder.*/logging_folder = \/opt\/mcm/' >> /opt/mcm/mcmd.conf.tmp
mv /opt/mcm/mcmd.conf.tmp /opt/mcm/mcmd.conf
```

Structure should be:
```
 /opt/mcm
         mcmd.conf
         mcm8.0.30
         mcm_data

 Folder mcm_data is where all data on disk will be stored by default.
 Folder mcmN.N.N contains installed version of MCM.
```

### Systemd configuration

##### Create user to run the MCM service and set file/folder permissions:
```
sudo useradd --no-create-home -s /bin/false mcm
sudo chown -R mcm:mcm /opt/mcm
chmod 600 /opt/mcm/mcmd.conf
```

##### Create systemd file:

Make sure to update path to correct version of mcm binaries in
mcm service file below.

Create file /etc/systemd/system/mcm.service like below.
```
[Unit]
Description=MySQL Cluster Manager
Documentation=https://dev.mysql.com/doc/mysql-cluster-manager/en/
After=network-online.target

[Service]
User=mcm
Group=mcm
Restart=always
Type=simple

ExecStart=/opt/mcm/mcm8.0.30/bin/mcmd --config=/opt/mcm/mcmd.conf

[Install]
WantedBy=multi-user.target
```

##### Start and enable service:
```
sudo systemctl daemon-reload
sudo systemctl start mcm
sudo systemctl enable mcm
sudo systemctl status mcm
```
If the service is not started correctly, look in messages file: ```sudo tail -150f /var/log/messages```

Remember also to set option StopOnError to 0 when creating your cluster so mcmd can restart failed data nodes ```set StopOnError:ndbmtd=0 mycluster;```.
