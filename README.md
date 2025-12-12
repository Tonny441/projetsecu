# projetsecu

# installation sur vps
## methode 1

INSTALLATION WAZUH SUR VM + Creation de VM
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
