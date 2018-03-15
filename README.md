# Stepup IRMA

A SAML Stepup Provider for IRMA

# Installation

Install dependencies (php, ntp, etc), eg for ubuntu 16.04:

	apt-get update
	apt-get upgrade -y
	apt install ntp unzip libxml2-utils -y

Install php and a web server, eg apache:

	apt install apache2 libapache2-mod-php composer php-xml php-mbstring -y
	a2enmod ssl
	service apache2 reload

Install certificates, eg from Let's Encrpyt:

	add-apt-repository -y ppa:certbot/certbot
	apt-get update
	apt install -y python-certbot-apache
	certbot --apache -d stepup-irma.example.org

Make sure certificates are (automatically) renewed.

Install stepup-irma:

	curl https://github.com/OpenConext/Stepup-irma/archive/vX.Y.tar.gz -sL | gunzip -c | tar -x -C /opt/ -f -
	cd /opt
	ln -s Stepup-irma-X.Y Stepup-irma
	cd Stepup-irma
	composer install

# Configuration

## Web server

edit your apache site config, eg `/etc/apache2/sites-enabled/000-default-le-ssl.conf` and sert the `DocumentRoot` with appropriate permissions, for instance:

	DocumentRoot /opt/Stepup-irma/www

	<Directory /opt/Stepup-irma/www>
		Options Indexes FollowSymLinks
                AllowOverride None
		FallbackResource index.php
		Require all granted
	</Directory>

Restart Apache

	service apache2 restart

## IRMA configuration

IRMA configuration is stored in the file `options.php`. Override settings in the file `local_options.php`.

You will need to use an IRMA API server to assist in obtaining and verifying IRMA attributes
Register with the IRMA API server to obtain a private key  and store it in a file, eg `irma_key.pem`.
Also obtain the certificate required to authenticate the IRMA API server and store it in a file, eg `apiserver.pem`.

Create the file `local_options.php` and include:

```php
<?php
$options['irma_api_server'] = 'https://irma.example.org';
$options['irma_web_server'] = 'https://privacybydesign.foundation/tomcat/irma_api_server';
$option['irma_keyid'] = "example";
$option['irma_issuer'] = "Example Organisation";
```

## SAML configuration

SAML configuration is stored in the file `config.php`. Override settings in the file `local_config.php`.

Generate SAML signing key (`key.pem`) and certificate (`cert.pem`).

	openssl req -x509 -sha256 -nodes -days 365 -subj '/CN=SAML signing' -newkey rsa:2048 -keyout key.pem -out cert.pem

retrieve the SAML signing certificate from the SP gateway and store it in a file `gateway_irma_sp.crt.pem`

	curl -s https://sa-gw.example.org/gssp/irma/metadata | \
	xmllint --xpath '//*[local-name()="X509Certificate"]/text()' - | \
	base64 -d | openssl x509 -inform der -out gateway_irma_sp.crt.pem

Create a file `local_config.php` and include:

```php
<?php
$config['sp']['https://sa-gw.test2.surfconext.nl/gssp/irma/metadata'] = array(
    'acs' => "https://sa-gw.example.org/gssp/irma/consume-assertion",
    'certfile' => dirname(__FILE__) . '/gateway_irma_sp.crt.pem',
);
```

# Testing

Instead of Apache you can also use the built-in web server of php 5.4+ and run from the command line:

        php -S ip:port -t www
