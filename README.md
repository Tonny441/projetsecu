# projetsecu

# installation sur vps
## methode 1

### INSTALLATION WAZUH SUR VM + Creation de VM
```bash
Mettre 8192mb de Base memory minimum
minimum 2 Cores
Si possible ~50Gb d’espace disk 

wazu vps
curl -sO https://packages.wazuh.com/4.14/wazuh-certs-tool.sh
curl -sO https://packages.wazuh.com/4.14/config.yml
indexer:

   - name: node-1

     ip: "<indexer-node-ip>"
changer ip pour propre ip
bash ./wazuh-certs-tool.sh -A --allow-public-ip
/home/moi/wazuh-install-files.tar
sudo bash ./wazuh-install.sh -a -i -p 8443
sudo ufw allow 1514/udp
sudo ufw allow 1515/tcp
sudo ufw allow 8443/tcp
sudo ufw reload
```
## methode 2
https://documentation.wazuh.com/current/installation-guide/index.html
https://documentation.wazuh.com/current/installation-guide/wazuh-indexer/step-by-step.html
```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-certs-tool.sh
curl -sO https://packages.wazuh.com/4.14/config.yml
nano  ./config.yml
### metre ip  ip: "<indexer-node-ip>" 192.168.56.2
bash ./wazuh-certs-tool.sh -A
tar -cvf ./wazuh-certificates.tar -C ./wazuh-certificates/ .
rm -rf ./wazuh-certificates
apt-get install debconf adduser procps
apt-get install gnupg apt-transport-https
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
apt-get update
apt-get -y install wazuh-indexer
nano /etc/wazuh-indexer/opensearch.yml
network.host metre ip 192.168.56.2
NODE_NAME=node-1
mkdir /etc/wazuh-indexer/certs
tar -xf ./wazuh-certificates.tar -C /etc/wazuh-indexer/certs/ ./$NODE_NAME.pem ./$NODE_NAME-key.pem ./admin.pem ./admin-key.pem ./root-ca.pem
mv -n /etc/wazuh-indexer/certs/$NODE_NAME.pem /etc/wazuh-indexer/certs/indexer.pem
mv -n /etc/wazuh-indexer/certs/$NODE_NAME-key.pem /etc/wazuh-indexer/certs/indexer-key.pem
chmod 500 /etc/wazuh-indexer/certs
chmod 400 /etc/wazuh-indexer/certs/*
chown -R wazuh-indexer:wazuh-indexer /etc/wazuh-indexer/certs
rm -f ./wazuh-certificates.tar
systemctl daemon-reload
systemctl enable wazuh-indexer
systemctl start wazuh-indexer
curl -k -u admin https://<WAZUH_INDEXER_IP_ADDRESS>:9200


```
