da sind erstmal nur schritte

# Einrichten

## Domain
1. zu domain registrarer gehen (wie digital plat) -> domain reservieren
2. zu nameserver dns anbieter gehen (wie cloudflare): DNS record: domain name auf ip von vps mappen (A-Record)
    - kann bis zu 24h dauern
    - gibt zwei nameserver adressen (ns1... und ns2...)
3. beide nameserveradressen von bspw cloudflare bei registrarer als nameserver eintragen
4. bei strato oder jeweiligem vps hoster domaine eintragen

## SSH
1. auf laptop:
`ssh-keygen -t rsa -b 4096 -C "cdrc.wnsch@gmail.com"` in .ssh ordner
2. `.pub` schlüssel datei zum server kopieren
3. als root neuen user erstellen:
    - `useradd cedric`
    - `passwd cedric` -> passwort setzen
    - `usermod -aG sudo cedric`
    - `su - cedric` -> nutzer wechseln
4. als user mit root rechten ssh key kopieren
    - in ssh `cd ~/.ssh`
```
sudo cat /root/.ssh/authorized_keys >> ~/.ssh/authorized_keys 
chmod 600 ~/.ssh/authorized_keys 
chown -R cedric:cedric ~/.ssh/
```

5. dann mit `kitty +kitten ssh name@ip`korekt von laptop aus verbinden

## firewall
- mit ufw
- `sudo ufw default deny incoming`
- `sudo ufw allow 22/tcp` --> ssh
- `sudo ufw allow 80/tcp` --> http
- `sudo ufw allow 443/tcp` --> https
- `sudo ufw allow in on podman1`
- `sudo ufw enable`
- `sudo ufw status verbose`
- port forwarding für containerports:
    - `sudo vi /etc/default/ufw `
    - dann `DEFAULT_FORWARD_POLICY="ACCEPT"` setzen (`DROP` lässt pakete verschwinden)

## Container
- `sudo apt install podman`
- `sudo apt install podman-compose`
- dann docker hub als registry hinzufügen:
    - `sudo vi /etc/containers/registries.conf`
    - ganz unten hinzufügen: `unqualified-search-registries = ["docker.io"]`

## SSL im vps-dinge repo
- bei nginx server port 80: nur den acme teil drinne lassen
- dann nginx (ohne certbot) starten 
- dannach:
```
sudo podman run --rm \
  --dns=1.1.1.1 \
  -v ./certbot/www:/var/www/certbot \
  -v ./certbot/conf:/etc/letsencrypt \
  docker.io/certbot/certbot certonly --webroot -w /var/www/certbot \
  -d micedric.dpdns.org --email cdrc.wnsch@gmail.com --agree-tos --no-eff-email
```
- dannach restliche nginx.conf wieder einbinden

## jenkins
- wenn jenkis nicht startet wegen permission denied im jenkins_home ordner: `sudo chown -R 1000:1000 ./jenkins_home`
=======
- dannach restliche nginx.conf wieder einbinden
