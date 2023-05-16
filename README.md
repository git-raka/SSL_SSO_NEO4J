# PART ONE = SSL NEO4J
Before ready assume you have a domain and pointing to cloudflare or anything 
<img width="680" alt="image" src="https://github.com/git-raka/SSL_Neo4j/assets/77326619/3d8cdde1-4a5c-4a87-8353-0c4c32cd1bdf">

``
sudo apt install certbot python3-certbot-apache
``
<img width="678" alt="image" src="https://github.com/git-raka/SSL_Neo4j/assets/77326619/18f64d3a-b401-434b-9163-89724c807342">
```
sudo systemctl stop apache2
sudo certbot certonly
```
<img width="678" alt="image" src="https://github.com/git-raka/SSL_Neo4j/assets/77326619/7bab54e7-b1be-40f0-9429-6b6ca73bc82e">
<img width="673" alt="image" src="https://github.com/git-raka/SSL_Neo4j/assets/77326619/f358f94d-4ebc-4190-8197-d23c9ee20ac6">
<img width="680" alt="image" src="https://github.com/git-raka/SSL_Neo4j/assets/77326619/7c3e33fa-508c-4b87-88bc-671654b8b33d">

```
sudo chgrp -R raka /etc/letsencrypt/*
sudo chmod -R g+rx /etc/letsencrypt/*
```

### Now create Shell Script for automation create dir and change permission
```
cd /home/raka/neo4j-enterprise-4.4.19/certificates
# Move old default stuff into a backup directory.
sudo mkdir bak
for certsource in bolt cluster https ; do
   sudo mv $certsource bak/
done
sudo mkdir bolt
sudo mkdir cluster
sudo mkdir https
export MY_DOMAIN=neo4j.devneo.cloud
for certsource in bolt cluster https ; do
   sudo ln -s /etc/letsencrypt/live/$MY_DOMAIN/fullchain.pem $certsource/neo4j.cert
   sudo ln -s /etc/letsencrypt/live/$MY_DOMAIN/privkey.pem $certsource/neo4j.key
   sudo mkdir $certsource/trusted
   sudo ln -s /etc/letsencrypt/live/$MY_DOMAIN/fullchain.pem $certsource/trusted/neo4j.cert ;
done
# Finally make sure everything is readable to the database
sudo chgrp -R raka *
sudo chmod -R g+rx *
```

### Now open neo4j conf
```
# BOLT Connector
dbms.connector.bolt.tls_level=REQUIRED
dbms.ssl.policy.bolt.enabled=true
dbms.ssl.policy.bolt.private_key=/home/raka/neo4j-enterprise-4.4.19/certificates/bolt/neo4j.key
dbms.ssl.policy.bolt.public_certificate=/home/raka/neo4j-enterprise-4.4.19/certificates/bolt/neo4j.cert
dbms.ssl.policy.bolt.client_auth=NONE

# HTTPS connector
dbms.connector.http.enabled=false
dbms.connector.https.enabled=true
dbms.ssl.policy.https.enabled=true
dbms.ssl.policy.https.client_auth=NONE
dbms.ssl.policy.https.private_key=/home/raka/neo4j-enterprise-4.4.19/certificates/https/neo4j.key
dbms.ssl.policy.https.public_certificate=/home/raka/neo4j-enterprise-4.4.19/certificates/https/neo4j.cert

# Directories
dbms.ssl.policy.bolt.base_directory=/home/raka/neo4j-enterprise-4.4.19/certificates/bolt
dbms.ssl.policy.https.base_directory=/home/raka/neo4j-enterprise-4.4.19/certificates/https
dbms.connector.https.listen_address=0.0.0.0:7473
dbms.connector.https.advertised_address=neo4j.devneo.cloud:7473

#OPTIONAL IF YOU NEED CLUSTER
dbms.ssl.policy.cluster.enabled=true
dbms.ssl.policy.cluster.base_directory=/home/neo4j/neo4j-enterprise-4.4.18/certificates/cluster/
dbms.ssl.policy.default.base_directory=/home/neo4j/neo4j-enterprise-4.4.18/certificates/cluster
dbms.ssl.policy.default.trusted_dir=/home/neo4j/neo4j-enterprise-4.4.18/certificates/cluster/trusted
dbms.ssl.policy.default.revoked_dir=/home/neo4j/neo4j-enterprise-4.4.18/certificates/cluster/revoked
dbms.ssl.policy.cluster.client_auth=NONE
dbms.ssl.policy.cluster.private_key=/home/neo4j/neo4j-enterprise-4.4.18/certificates/cluster/neo4j.key
dbms.ssl.policy.cluster.public_certificate=/home/neo4j/neo4j-enterprise-4.4.18/certificates/cluster/neo4j.cert
```
# PART TWO = SINGLE SIGN ON NEO4J
```
dbms.security.authentication_providers=oidc-google
dbms.security.authorization_providers=oidc-google,native
dbms.security.oidc.google.display_name=Google
dbms.security.oidc.google.auth_flow=pkce
dbms.security.oidc.google.well_known_discovery_uri=https://accounts.google.com/.well-known/openid-configuration
dbms.security.oidc.google.audience=264800647591-h8m386l9ntuntirps9i0bf9g8ig3hrvu.apps.googleusercontent.com
dbms.security.oidc.google.claims.username=email
dbms.security.oidc.google.params=client_id=264800647591-h8m386l9ntuntirps9i0bf9g8ig3hrvu.apps.googleusercontent.com;response_type=code;scope=openid profile email
dbms.security.oidc.google.token_params=client_secret=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
dbms.security.oidc.google.config=token_type_principal=id_token;token_type_authentication=id_token
```

<img width="823" alt="image" src="https://github.com/git-raka/SSL_Neo4j/assets/77326619/5c508d14-a0a4-42a7-874c-5fc8853713d1">

