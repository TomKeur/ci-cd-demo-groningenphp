# Continuous Integration en Continuous Deployment demo

This README is in Dutch since the presentation was held in Dutch.

**Presentatie werd gegeven door:**

* Frank Klein Koerkamp
* Tom Keur

Het verhaal dat we verteld hebben tijdens de presentatie is gebaseerd op onze ervaringen die we hebben opgedaan tijdens ons werk bij [Webstores Digital Partner](https://www.webstores.nl)

Aangezien ik (Tom Keur) de demo heb gegeven, ben ik hieronder verder gegaan in de ik vorm.

In deze repository staan alle configuraties zoals ik deze heb gebruikt tijdens de demo van onze presentatie: "Continuous Integration & Deployment".

Ik ben er tijdens de voorbereiding vanuit gegaan, dat het gaat om een "traditionele" manier van hosten gaat, en dus niet om een containeromgeving. Dit veranderd eigenlijk niet heel erg veel aan het hele CI/CD proces. Het grote verschil is dat je artifact ook je container (inclusief source) is, en dat je niet deployed via Ansistrano.

Mocht je nog vragen hebben over de demo dan kun je mij beste bereiken via Twitter: [@TomKeur](https://twitter.com/tomkeur).

Gebruik gemaakt van (alles is open source):

* [Scripts to Rule them All (eigen PHP Variant)](https://github.com/github/scripts-to-rule-them-all)
* [Ansistrano Deploy](https://github.com/ansistrano/deploy)
* [Jenkins Blue Ocean](https://jenkins.io/projects/blueocean/)
* [Vagrant](https://www.vagrantup.com/)
* [VirtualBox](https://www.virtualbox.org/)
* [Docker](https://www.docker.com/get-started)
* [docker-compose](https://docs.docker.com/compose/)
* [Symfony demo application](https://github.com/symfony/demo)

## Symfony demo application

Ik heb de Symfony demo application geïnstalleerd via het volgende commando:

```bash
composer create-project symfony/symfony-demo symfony-demo
```

Ik heb de Symfony demo application in een losse repository gezet, omdat hier ook de `Jenkinsfile`, `Scripts to Rule them All` en `Ansistrano` configuratie staat. Voor alle zekerheid heb ik deze ook in deze "algemene" repository toegevoegd, maar de [Symfony-demo repository is hier te vinden](https://github.com/TomKeur/symfony-demo).
Op deze manier kun je hem eenvoudiger toevoegen in Jenkins omdat de Jenkinsfile dan in de root van het Git project staat.

## Vagrant

Virtual machine aangemaakt met Ubuntu 18.04 op VirtualBox, prima geschikt voor demo/ontwikkel doeleinden.
Op deze machine heb ik de volgende software geïnstalleerd:

* Nginx
* PHP 7.2 FPM

Aangezien het om een demo gaat, heb ik geen provision script gebruikt, maar met het volgende command kun je alle software installeren:

```bash
sudo apt update &&
sudo apt install nginx -y --no-install-recommends \
php7.2-cli \
php7.2-fpm \
php7.2-xml \
php7.2-intl \
php7.2-mbstring
```

### Instellingen Virtual machine

#### Nginx configuratie

Ik heb direct de default vhost van Nginx aangepast, deze is hier te vinden: `/etc/nginx/sites-enabled/default`

```nginx
# Default server configuration
#
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/vhosts/demo.groningenphp.test/public_html;

    # Add index.php to the list if you are using PHP
    index index.php index.html index.htm index.nginx-debian.html;

    server_name _;

    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        try_files $uri /index.php$is_args$args;
    }

    # pass PHP scripts to FastCGI server
    location ~ ^/index\.php(/|$) {
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        include snippets/fastcgi-php.conf;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        fastcgi_param DOCUMENT_ROOT $realpath_root;
        internal;
    }
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
    }
    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    location ~ /\.ht {
        deny all;
    }

    error_log /var/log/nginx/demo.groningenphp.test_error.log;
    access_log /var/log/nginx/demo.groningenphp.test_access.log;
}

```

#### PHP-FPM configuratie

PHP-FPM configuratie, deze is te wijzigen in: `/etc/php/7.2/fpm/pool.d/www.conf`

Om geen problemen te krijgen met permissies draai ik PHP FPM onder de default Vagrant user, op deze manier kan ik ook zonder problemen een deploy uitvoeren met mijn Vagrant user.

Aangezien het een demo betreft, heb ik onderaan in `/etc/php/7.2/fpm/pool.d/www.conf`, nog wat harde environment variables gezet.

```ini
; Unix user/group of processes
; Note: The user is mandatory. If the group is not set, the default user's group
;       will be used.
user = vagrant
group = vagrant

; Hackfix for environment variables for Symfony :)
env["APP_ENV"] = prod
env["APP_DEBUG"] = 0
env["APP_SECRET"] = "xxx"
env["DATABASE_URL"] = "sqlite:///%kernel.project_dir%/data/database.sqlite"
env["MAILER_URL"] = "MAILER_URL=null://localhost"
```

#### Bash aliases

Aangezien je tijdens het configureren vaak dezelfde commando's moet uitvoeren heb ik hier bash aliases voor gemaakt.

Deze staan in `~/.bashrc`.

```bash
alias restartphp="sudo systemctl restart php7.2-fpm.service"
alias restartnginx="sudo systemctl restart nginx.service"
alias fpmlog="sudo tail -f /var/log/fpm-php.www.log"
alias nginxaccess="sudo tail -f /var/log/nginx/demo.groningenphp.test_access.log"
alias nginxerror="sudo tail -f /var/log/nginx/demo.groningenphp.test_error.log"

export APP_ENV="prod"
```

## Jenkins

Om Jenkins te starten dien je Docker en docker-compose te hebben geïnstalleerd, vervolgens kun je het volgende commando gebruiken.

```bash
docker-compose up -d
```

Wanneer je Jenkins voor de eerste keer draait heb je nog geen wachtwoord, dit kun je opvragen doormiddel van:

```bash
docker-compose exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Ik heb gebruik gemaakt van de Jenkins Blue Ocean image van [Docker hub](https://hub.docker.com/r/jenkinsci/blueocean).

## Scripts to Rule Them All

In mijn demo maakte ik ook gebruik van Scripts To Rule Them All, in deze bashscripts heb ik gebruik maakt van een PHP Docker image, deze is toegevoegd in de `php72-docker-file`. Hij is ook rechtstreeks te gebruiken via [Docker Hub](https://hub.docker.com/r/tomkeur/php72) 

**Waarschuwing:** Aangezien deze image alleen tijdens de demo is gebruikt onderhoud ik deze niet verder. Het is verstandig om dus een andere Docker image te gebruiken. Ik heb hem alleen toegevoegd, zodat het een compleet werkende demo is.