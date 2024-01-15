# Installing Sentry with docker-compose.

## Quick Installation

**Before starting the instance, direct the domain (subdomain) to the ip of the server where Sentry will be installed!**

## 1. Sentry Server Requirements
From and more
- 4 CPUs 
- 11 RAM 
- 20 Gb 

Run for Ubuntu 22.04

``` bash
sudo apt-get purge needrestart
```

Install docker and docker-compose:

``` bash
curl -s https://raw.githubusercontent.com/6Ministers/sentry-docker-compose-fast-deploy/master/setup.sh | sudo bash -s
```

If `curl` is not found, install it:

``` bash
$ sudo apt-get install curl
# or
$ sudo yum install curl
```


Download Sentry repository:

``` bash
git clone https://github.com/getsentry/self-hosted.git
```

Go to the catalog
``` bash
cd self-hosted
```

Execute the command

(Проверьте последню версию в репозитории https://github.com/getsentry/self-hostedhttps://github.com/getsentry/self-hosted
)
``` bash
git checkout 23.12.1
```

Скачайте `Caddyfile`
``` bash
wget https://raw.githubusercontent.com/6Ministers/sentry-docker-compose-fast-deploy/master/Caddyfile
```

In the configuration file `Caddyfile`, set the following parameters. 

Specify your domain (subdomain):
https://your-domain.com

``` bash
https://your-domain.com:443 {
    reverse_proxy 127.0.0.1:9000
	encode zstd gzip
	file_server
	
	# Secure headers, all from .htaccess except Permissions-Policy, STS and X-Powered-By
	header {
		Strict-Transport-Security max-age=31536000
		Permissions-Policy interest-cohort=()
		X-Content-Type-Options nosniff
		X-Frame-Options SAMEORIGIN
		Referrer-Policy no-referrer
		X-XSS-Protection "1; mode=block"
		X-Permitted-Cross-Domain-Policies none
		X-Robots-Tag "noindex, nofollow"
		-X-Powered-By
}
	
}
```


Add to docker-compose.yml after `services:`
``` bash
services:
  caddy:
    image: caddy:alpine
    restart: unless-stopped
    container_name: caddy
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - ./certs:/certs
      - ./config:/config
      - ./data:/data
      - ./sites:/srv
    network_mode: "host"
```


Run the script
``` bash
./install.sh --skip-user-prompt --no-report-self-hosted-issues
```


Флаги:

`--skip-user-prompt` — пропустить запрос создания пользователя (создадим его отдельной командой, так будет проще).
`--no-report-self-hosted-issues` — пропустить запрос для анонимной отправки данных разработчикам Sentry с вашего хоста (помогает разработчикам улучшать продукт, но немного отъедает ресурсов; решите для себя, нужно ли вам это).
Начинается процесс проверки соответствия требованиям и скачивание необходимых образов (docker pull).

После будет выведено сообщение, что можно запустить Sentry
``` bash
You're all done! Run the following command to get Sentry running:
docker-compose up -d
```


**Run Sentry:**

``` bash
docker-compose up -d
```


Создадим пользователя

``` bash
docker-compose run --rm web createuser
```

Then open `https://your-domain.com` to access Sentry

## Sentry container management

**Run Sentry**:

``` bash
docker-compose up -d
```

**Restart**:

``` bash
docker-compose restart
```

**Restart**:

``` bash
sudo docker-compose down && sudo docker-compose up -d
```

**Stop**:

``` bash
docker-compose down
```

## Documentation
https://github.com/getsentry/self-hosted

https://timeweb.cloud/tutorials/servers/sentry-monitoring-i-otslezhivanie-oshibok
