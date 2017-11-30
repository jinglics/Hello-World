# Step 1: Install mongodb on three servers
```bash
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
sudo apt-get update && apt-get install -y mongodb-org
```

# Step 2: Start mongod service
<!-- replace bingIp with the machine ip for each data server -->
```bash
mkdir -p /var/run/mongodb
chown -R mongodb:mongodb /var/run/mongodb
echo -e "# mongod.conf\nsystemLog:\n  destination: file\n  logAppend: true\n  path: /var/log/mongodb/mongod.log\nstorage:\n  dbPath: /var/lib/mongodb\n  journal:\n    enabled: true\nprocessManagement:\n  fork: true  # fork and run in background\n  pidFilePath: /var/run/mongodb/mongod.pid  # location of pidfile\nnet:\n  port: 27017\n  bindIp: 10.160.0.2  # Listen to local interface only, comment to listen on all interfaces.\nreplication:\n  oplogSizeMB: 2000\n  replSetName: coinut\n" > /etc/mongod.conf
mongod -f /etc/mongod.conf
```

# Step 3: Configure mongodb from master mongo shell  
```bash
mongo --host master_ip
> use admin
> config = {_id: "coinut", members: [{_id: 172, host: "[master_ip]:27017"},{_id: 173, host: "[slave1_ip]:27017"}, {_id: 174, host: "[slave2_ip]:27017"}]}
> rs.initiate(config)
> rs.status()
mongo --host slave1_ip
> rs.slaveOk()
mongo --host slave2_ip
> rs.slaveOk()
/etc/init.d/mongod stop && /usr/bin/mongod -f /etc/mongod.conf
```

