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

## Firewall
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
- rootles portbinding ab port 80: `sudo sysctl net.ipv4.ip_unprivileged_port_start=80`
- podman für user aktivieren: `systemctl --user enable --now podman.socket`
- und netzwerk global erstellen: `podman network create mynet`

## SSL im vps-dinge repo
- bei nginx server port 80: nur den acme teil drinne lassen
- dann nginx (ohne certbot) starten 
- dannach:
```
podman run -it --rm   --network mynet   -v ./certbot/conf:/etc/letsencrypt   -v ./certbot/www:/var/www/certbot   certbot/certbot certonly --manual --preferred-challenges dns   --cert-name micedric.dpdns.org --expand   -d micedric.dpdns.org -d '*.micedric.dpdns.org'
```
- dannach restliche nginx.conf wieder einbinden
- eventuell hash summen als cloudflare record TXT eintragen, um domain zu verifizieren

## BasicAuth im repo
- `sudo apt install -y apache2-utils`
- `htpasswd -c ./nginx/auth/martin.htpasswd martin`

## Jenkins
- wenn jenkis nicht startet wegen permission denied im jenkins_home ordner: `sudo chown -R cedric ./jenkins_home`
=======
- dannach restliche nginx.conf wieder einbinden
- für ssh aus jenkins container raus: 
```
ssh-keygen -t ed25519 -f ~/.ssh/jenkins_deploy -N ""
cat ~/.ssh/jenkins_deploy.pub >> ~/.ssh/authorized_keys
```
- in jenkins: → Manage Jenkins → Credentials → System → Global credentials → Add Credentials
- username: von host (cedric), private key: .ssh/jenkins_deploy cat eintragen
- SSH Agent Plugin installieren



### Pipeline einrichten (von Anfang an)
Zunächst muss ein Jenkins Server eingerichtet werden. Für diesen sollte diese Grundlage verwendet werden. Es wird angenommen, dass ein vorgeschalteter HTTP-Server existiert, der Anfragen mittels Reverse-Proxy weiterleitet.

Dann wird ein Admin-Account angelegt.

Um Jenkins Zugriff auf die relevanten Repos zu gestatten muss eine GitHub App eingerichtet werden. 

- GitHub.com öffnen
- Profilbild oben rechts wählen
- Settings wählen
- Developer settings wählen
- auf vorausgewähltem GitHub Apps Tab bleiben oder zu diesem wechseln
- New GitHub App wählen
- Sinnvollen GitHub App name wählen (z.B. Jenkins-WS25Teamprojekt; Achtung: Dieser muss global eindeutig sein)
- Homepage URL (beliebig z.B. URL der Jenkins-Instanz)
- Callback URL (beliebig, da keine Verwendung)
- Webhook Active abgeschaltet lassen (betreffen Events bezüglich der Anwendung selbst)
- Permissions hinzufügen (ausschließlich Repository permissions notwendig):
- Checks 
    - (Read and write)
    - Contents (Read-only)
    - Webhooks (Read and write)
- Scope der Anwendung (Where can this GitHub App be installed) auf "Any account" stellen, um in der Organisation installieren zu können
- Create GitHub App wählen

Es muss außerdem ein Client secret und ein private key generiert werden um Jenkins zu erlauben im Namen dieser Anwendung zu handeln. Diese (sowie die App ID) muss Jenkins hinzugefügt werden.

- Jenkins Einstellungen öffnen
- Credentials wählen
- System wählen
- Global credentials (unrestriced) wählen
- Add Credentials wählen
- GitHub App als Kind auswählen
- ID wählen (z.B. github-app)
- App ID mit App ID aus GitHub Seite füllen
- Private Key einfügen
- Konvertieren mittels `openssl pkcs8 -topk8 -inform PEM -outform PEM -in foobar.private-key.pem -out new-key.pem -nocrypt`
    - korrektes Format:

        -----BEGIN PRIVATE KEY-----
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXXXXXXXXXXXXXXX
        -----END PRIVATE KEY-----

Nun kann die App den relevanten Repositories hinzugefügt (installiert) werden:

- Zu der App auf GitHub navigieren (siehe oben)
- Install App Tab wählen
- Organisation wählen und Install oder Zahnrad wählen
- Repositories auswählen und bestätigen

Damit Jenkins Updates von GitHub erhalten kann muss der sog. WebHook-Mechanismus eingerichtet werden. Hierfür muss ein Credential generiert (zum Beispiel mit einem Passwortgenerator) und sowohl in Jenkins als auch im GitHub Repository eingestellt werden.

Zunächst die Einrichtung unter Jenkins:
- Jenkins Einstellungen öffnen
- Credentials wählen
- System wählen
- Global credentials (unrestriced) wählen
- Add Credentials wählen
- Secret text als Kind wählen
- Secret eintragen
- ID wählen (z.B. github-webhook)
- Create wählen

Nun muss dieses Secret als WebHook-Secret eingestellt werden:
- Jenkins Einstellungen öffnen
- System wählen
- zu GitHub scrollen
- Advanced aufklappen
- Unter Shared secret das vorher erstellte GitHub WebHook Secret wählen
- Signature algorithm auf SHA-256 stellen

Nun kann WebHooks für einzelne Jobs eingerichtet werden.

Um einen Job automatisch durch pushes zu starten, muss der Haken bei GitHub project gesetzt und die URL gesetz werden (z.B. https://github.com/WS25Teamprojekt/mkdocs_internal_stundenplan.git)

Zusätzlich muss ein Haken unter Triggers bei "GitHub hook trigger for GITscm polling" gesetzt werden.

Außerdem müssen die App Credentials eingestellt werden, um Lesezugriff auf private Repos zu erlauben.

Hierfür muss unter Pipeline -> Definition "Pipeline script from SCM" eingestellt werden. Unter SCM Git wählen, die Repository URL eintragen (z.B. https://github.com/WS25Teamprojekt/mkdocs_external_stundenplan.git) und schließlich die App Credentials gewählt werden. Wenn der Hauptbranch nicht master sonder main heißt muss unter Branches to build */master zu */main abgeändert werden.

Außerdem muss eventuell der Script Path geändert werden, wenn die Jenkinsdatei nicht im Root-Verzeichnis liegt und Jenkinsfile heißt.

Der letzte Schritt besteht darin die Webhooks im Repository einzurichten. Hierfür muss in den Einstellungen des Repo der Tab Webhooks gewählt werden und ein neuer Webhook hinzugefügt werden. Es muss die Payload URL eingetragen werden (z.B. https://sturmkeks.de/jenkins/github-webhook/). Der Content type bleibt auf application/x-www-form-urlencoded stehen und das Secret muss eingetragen werden.

Außerdem sollte SSL verification eingeschaltet werden.

Für trigger events reicht "Just the push event".

Der Webhook muss auf "Active" gesetzt werden und hinzugefügt werden.

Nach Abschluss dieser Schritte sollte Jenkins nun in der Lage sein Jobs automatisch bei push anzustoßen und die Ergebnisse rückzumelden.