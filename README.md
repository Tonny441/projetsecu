# projetsecu
#Procédure de déploiement


IP ServeurDirection => 192.168.56.3
IP ServeurDeveloppeur => 192.168.56.4
IP ServeurRessourceHumaine => 192.168.56.5


Vm Développeur  : accès aux projets, code source, documentation technique, parfois aux données clients (selon le projet). 

Ressources humaines : accès aux dossiers du personnel, bulletins de salaire, contrats, données confidentielles. 


 Direction & Managers : accès global aux documents stratégiques, financiers, RH et client. 

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
## indexer
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
## manager
https://documentation.wazuh.com/current/installation-guide/wazuh-server/step-by-step.html
```bash
apt-get install gnupg apt-transport-https
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
apt-get update
apt-get -y install wazuh-manager
apt-get -y install filebeat
curl -so /etc/filebeat/filebeat.yml https://packages.wazuh.com/4.14/tpl/wazuh/filebeat/filebeat.yml
nano /etc/filebeat/filebeat.yml metre host ip 192.168.56.2
filebeat keystore create
echo admin | filebeat keystore add username --stdin --force
echo admin | filebeat keystore add password --stdin --force
curl -so /etc/filebeat/wazuh-template.json https://raw.githubusercontent.com/wazuh/wazuh/v4.14.1/extensions/elasticsearch/7.x/wazuh-template.json
chmod go+r /etc/filebeat/wazuh-template.json
curl -s https://packages.wazuh.com/4.x/filebeat/wazuh-filebeat-0.4.tar.gz | tar -xvz -C /usr/share/filebeat/module
NODE_NAME=wazu-manager
mkdir /etc/filebeat/certs
tar -xf ./wazuh-certificates.tar -C /etc/filebeat/certs/ ./$NODE_NAME.pem ./$NODE_NAME-key.pem ./root-ca.pem
mv -n /etc/filebeat/certs/$NODE_NAME.pem /etc/filebeat/certs/filebeat.pem
mv -n /etc/filebeat/certs/$NODE_NAME-key.pem /etc/filebeat/certs/filebeat-key.pem
chmod 500 /etc/filebeat/certs
chmod 400 /etc/filebeat/certs/*
chown -R root:root /etc/filebeat/certs
echo '<INDEXER_USERNAME>' | /var/ossec/bin/wazuh-keystore -f indexer -k username
echo '<INDEXER_PASSWORD>' | /var/ossec/bin/wazuh-keystore -f indexer -k password
/var/ossec/etc/ossec.conf metre ip a host  <host>https://0.0.0.0:9200</host>
systemctl daemon-reload
systemctl enable wazuh-manager
systemctl start wazuh-manager
systemctl status wazuh-manager
systemctl daemon-reload
systemctl enable filebeat
systemctl start filebeat
```
## dashboard
https://documentation.wazuh.com/current/installation-guide/wazuh-dashboard/step-by-step.html
```bash
apt-get install debhelper tar curl libcap2-bin 
apt-get install gnupg apt-transport-https
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
apt-get update
apt-get -y install wazuh-dashboard
/etc/wazuh-dashboard/opensearch_dashboards.yml server host metre 192.168.56.2
NODE_NAME=wazu-dashboard
mkdir /etc/wazuh-dashboard/certs
tar -xf ./wazuh-certificates.tar -C /etc/wazuh-dashboard/certs/ ./$NODE_NAME.pem ./$NODE_NAME-key.pem ./root-ca.pem
mv -n /etc/wazuh-dashboard/certs/$NODE_NAME.pem /etc/wazuh-dashboard/certs/dashboard.pem
mv -n /etc/wazuh-dashboard/certs/$NODE_NAME-key.pem /etc/wazuh-dashboard/certs/dashboard-key.pem
chmod 500 /etc/wazuh-dashboard/certs
chmod 400 /etc/wazuh-dashboard/certs/*
chown -R wazuh-dashboard:wazuh-dashboard /etc/wazuh-dashboard/certs
systemctl daemon-reload
systemctl enable wazuh-dashboard
systemctl start wazuh-dashboard
/usr/share/wazuh-dashboard/data/wazuh/config/wazuh.yml metre ip dans url
/usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh --api --change-all --admin-user wazuh --admin-password wazuh
```



# clien 
Création des VMs 
(Créé 3 VM, changer le hostname et ip)

Sur les VM : sudo apt update && sudo apt upgrade -y


Sur CHAQUE VM



# --- Création utilisateur + groupe ---
```bash
sudo adduser edan
sudo groupadd direction
sudo usermod -aG direction edan
```
# --- Arborescence des dossiers ---
```bash
sudo mkdir -p /docs/public
sudo mkdir -p /docs/interne
sudo mkdir -p /docs/confidentiel
sudo mkdir -p /docs/secret
```
# --- Fichiers exemples ---
```bash
sudo touch /docs/public/rapport_annuel.pdf
sudo touch /docs/interne/guide_employe.docx
sudo touch /docs/confidentiel/base_clients.xlsx
sudo touch /docs/secret/plan_fusion.docx
```
# --- Permissions ---
```bash
# Direction accès total sur "secret"
sudo chown edan:direction /docs/secret
sudo chmod 770 /docs/secret

# Direction lecture/écriture sur "confidentiel"
sudo chown edan:direction /docs/confidentiel
sudo chmod 770 /docs/confidentiel

# Lecture/écriture partagée sur "interne"
sudo chown edan:direction /docs/interne
sudo chmod 775 /docs/interne

# Tout le monde lecture sur "public"
sudo chown edan:direction /docs/public
sudo chmod 755 /docs/public
```
# --- Installation Wazuh Agent ---
```bash
sudo apt-get install gnupg apt-transport-https

curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH \
| sudo gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import

sudo chmod 644 /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" \
| sudo tee -a /etc/apt/sources.list.d/wazuh.list

sudo apt-get update
sudo apt-get install wazuh-agent
```
# --- Configuration agent Wazuh ---
```bash
sudo nano /var/ossec/etc/ossec.conf

# INSÉRER :
# <client>
#   <server>
#     <address>IP_DU_SERVEUR_WAZUH</address>
#     <protocol>tcp</protocol>
#     <port>1514</port>
#   </server>
# </client>
```
# --- Activer et démarrer l’agent ---
```bash
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
sudo systemctl status wazuh-agent
```








# --- Création utilisateur + groupe ---
```bash
sudo adduser rh1
sudo groupadd rh
sudo usermod -aG rh rh1
```
# --- Arborescence des dossiers ---
```bash
sudo mkdir -p /docs/public
sudo mkdir -p /docs/interne
sudo mkdir -p /docs/confidentiel
sudo mkdir -p /docs/secret
```
# --- Fichiers exemples ---
```bash
sudo touch /docs/interne/politique_conges.docx
sudo touch /docs/confidentiel/dossier_employes.xlsx
```
# --- Permissions ---
```bash
# RH : Accès lecture/écriture sur "confidentiel"
sudo chown rh1:rh /docs/confidentiel
sudo chmod 770 /docs/confidentiel

# RH : Accès lecture/écriture sur "interne"
sudo chown rh1:rh /docs/interne
sudo chmod 775 /docs/interne

# Public : lecture pour tous
sudo chmod 755 /docs/public

# Aucune permission sur "secret" (la direction contrôle)
sudo chmod 700 /docs/secret
```
# --- Installation Wazuh Agent ---
```bash
sudo apt-get install gnupg apt-transport-https

curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH \
| sudo gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import

sudo chmod 644 /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" \
| sudo tee -a /etc/apt/sources.list.d/wazuh.list

sudo apt-get update
sudo apt-get install wazuh-agent
```
# --- Configuration agent ---
```bash
sudo nano /var/ossec/etc/ossec.conf

# INSÉRER :
# <client>
#   <server>
#     <address>IP_DU_SERVEUR_WAZUH</address>
#     <protocol>tcp</protocol>
#     <port>1514</port>
#   </server>
# </client>

sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
sudo systemctl status wazuh-agent
```
# --- Création utilisateur + groupe ---
```bash
sudo adduser dev1
sudo groupadd dev
sudo usermod -aG dev dev1
```
# --- Arborescence des dossiers ---
```bash
sudo mkdir -p /docs/public
sudo mkdir -p /docs/interne
sudo mkdir -p /docs/confidentiel
sudo mkdir -p /docs/secret
sudo mkdir -p /docs/devtools
```
# --- Fichiers exemples ---
```bash
sudo touch /docs/interne/guide_technique.md
sudo touch /docs/devtools/api_keys.txt
```
# --- Permissions ---
```bash
# DEV : Lecture/écriture sur "interne"
sudo chown dev1:dev /docs/interne
sudo chmod 775 /docs/interne

# DEV : Lecture/écriture uniquement dans leur espace "devtools"
sudo chown dev1:dev /docs/devtools
sudo chmod 770 /docs/devtools

# DEV : Pas d’accès à confidentiel
sudo chmod 700 /docs/confidentiel

# DEV : Pas d’accès à secret
sudo chmod 700 /docs/secret

# Public : lecture pour tous
sudo chmod 755 /docs/public
```
# --- Installation Wazuh Agent ---
```bash
sudo apt-get install gnupg apt-transport-https

curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH \
| sudo gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import

sudo chmod 644 /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" \
| sudo tee -a /etc/apt/sources.list.d/wazuh.list

sudo apt-get update
sudo apt-get install wazuh-agent
```
# --- Configuration agent ---
```bash
sudo nano /var/ossec/etc/ossec.conf

# INSÉRER :
# <client>
#   <server>
#     <address>IP_DU_SERVEUR_WAZUH</address>
#     <protocol>tcp</protocol>
#     <port>1514</port>
#   </server>
# </client>

sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
sudo systemctl status wazuh-agent
```


# Problèmes rencontrés (difficultés imprévu): 

## Imprévus (Emeric)
Pas assez de RAM sur le premier VPS 

Problèmes: Impossible d’installer Wazuh sur ce VPS

Solution: Obligé de changer de VPS pour l’installation de Wazuh 
Rendu du compte que Wazuh demandait 50GB disk dur et le VPS n’était pas assez fort. Donc switch sur une VM



## Problème: Mes VM (ludovic) font des erreurs lorsque j’essai d’installer wazuh-agents 

Problèmes: Le sudo apt-get update et la commande d’installation font une erreur bizarre qui empêche d’installer

Solution: Commande sur le reddit => Rien changer
Solution2: recommencer la VM => Rien changer
Solution 3: demander à Thibault de les faire => Ça a marché :)


https://askubuntu.com/questions/298177/a-failed-to-fetch-error-occurs-when-apt-get-update-is-run-how-do-i-fix-this


résolution => Donc selon des sources externes, le premier essai pour installer Wazuh sur ma VM a été la corruption des sources APT du  dépôt Wazuh (clés GPG expirées ou repository mal reconnue). Cela faisait que même en recommençant l’installation Waouh plantait




## Imprévu Installation Wazuh sur VM (ludovic & Thibault)

Imprévus: Les commandes d’installation de wazuh-indexer, manager, dashboard 
gelait systématiquement à 20% de l’installation.

Problème: empêche l’installation de Wazuh

Solution 1: Nettoyer complètement les paquets Wazuh et refait une installation propre
Solution 2: Augmentation de CPU 1 => 2

Solution 3 : ChatGPT dit de tester ca
Si une installation est gelée, dpkg reste souvent locké.
```bash
sudo lsof /var/lib/dpkg/lock
sudo lsof /var/lib/apt/lists/lock
sudo lsof /var/cache/apt/archives/lock
```
Si quelque chose apparaît → tue-le :
```bash
sudo kill -9 PID
```
Puis :
```bash
sudo rm /var/lib/dpkg/lock
sudo rm /var/lib/apt/lists/lock
sudo rm /var/cache/apt/archives/lock
sudo dpkg --configure -a
sudo apt --fix-broken install -y
```
Ca a rien changé pantoute

Finalement Emeric a réussi a créé une VM avant donc j’ai abandonné..






Imprévu: Manque de RAM pour lancer les 3 VMs d’environnement + VM Wazuh (Emeric)

Recommandé 8 coeurs 16 RAM pour chaque service PS: IL EN A 4!!

Jamais pensé que ca pourrait être autant lourd donc on est fourré.


Solutions: Augmenter la RAM de la machine hôte
(> 16 Go recommandé si tu veux 3 VMs + Wazuh)
Réduire la RAM des VMs non essentielles
Ex : passer chaque VM à 1.5–2 Go.

Solution 2: Parce que la RAM autorisé par défaut par Wazuh est de 1G, mais il faut plus. 
PS: C’est cave de mettre par défaut de la RAM alors que tu c'est que ton logiciel à besoin de plus, - 3h de travail pour rien…
https://chatgpt.com/c/693c1c5e-0844-8333-933c-476f7757aa64



PROBLEME: Wazuh demande certificat (IP publique/privé) (Emeric & Ludovic)


Le script demande une IP privée, mais les ip du vps son public 


Et Wazuh refuse de démarrer parce que le certificat ne correspond pas

Solution : Générer de NOUVEAUX certificats avec TOUTES tes IP + ton hostname

MAIS, impossible de générer les nouveaux certificats, car pas assez de disque dur comme sur le problème d’avant (boucle infini)


https://chatgpt.com/c/693c179d-80f8-8328-8df0-5f7b72af08b2

https://documentation.wazuh.com/current/user-manual/wazuh-server-cluster/adding-new-server-nodes/certificates-creation.html




Problème: Il faut un nom de domaine liés à l’ip et vu que c'est une VM ca marche pas (Pour pouvoir utiliser le dashboard) (Thibault)


Solution:  Il n'y en a pas. Impossible de mettre un nom de domaine avec une IP sur une VM.








