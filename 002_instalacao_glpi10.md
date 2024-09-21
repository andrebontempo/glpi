## 1 Instalação GLPI 10 by Lucas Levi – 2024

### Passo 1 : Acessar:

```bash
cd /var/www
```
### 1.1 Baixar glpi : 
```bash
wget https://github.com/glpi-project/glpi/archive/refs/tags/10.0.10.zip
```
Referência : https://github.com/glpi-project/glpi/releases/

### 1.2 Descompactar pasta:
```bash
unzip 10.0.10.zip
```
### 1.3 Renomear pasta:
```bash
mv glpi-10.0.10 glpi
```
### Passo 2
### 2.1 Após descompactar a pasta, acessar 
```bash
cd /inc
```  
e criar o arquivo 
```bash
nano downstream.php 
```
e setar esse conteúdo :
```bash
<?php
define('GLPI_CONFIG_DIR', '/etc/glpi/');
if (file_exists(GLPI_CONFIG_DIR . '/local_define.php')) {
require_once GLPI_CONFIG_DIR . '/local_define.php';
}
```
### 2.2 Criar os diretórios a seguir :
```bash
mkdir /etc/glpi
```
```bash
mkdir /var/lib/glpi
```
```bash
mkdir /var/log/glpi
```
### 2.3 Criar sub diretórios da pasta files
```bash
cd /var/www/glpi/files
```
```bash
cp * -Rf /var/lib/glpi
```
### 2.4 Dar permissões do usuário apache nessas pastas
```bash
chown www-data:www-data /etc/glpi -Rf
```
```bash
chown www-data:www-data /var/lib/glpi -Rf
```
```bash
chown www-data:www-data /var/log/glpi -Rf
```
```bash
chown www-data:www-data /etc/www/glpi -Rf
```
### 2.5 Em 
```bash
cd /etc/glpi 
```
criar o arquivo 
```bash
nano local_define.php
```
 com o seguinte conteúdo :
```bash
<?php
define('GLPI_VAR_DIR', '/var/lib/glpi');
define('GLPI_LOG_DIR', '/var/log/glpi');
```
### 2.6 Habilitar diretiva ‘session.cookie_httponly’ no PHP
Caminho 1 : 
```bash
nano /etc/php/8.2/cli/php.ini
```
Ctrl + W, session.cookie_httponly, session.cookie_httponly = ON
Ctrl + O, Ctrl + X
Caminho 2 : 
```bash
nano /etc/php/8.2/apache2/php.ini
```
Ctrl + W, session.cookie_httponly, session.cookie_httponly = ON
Ctrl + O, Ctrl + X
```bash
systemctl restart apache2
```
### 2.7 Ativar cron do GLPI
```bash
nano /etc/crontab
```
adicionar a linha :
```bash
* * * * * root php -f /var/www/glpi/front/cron.php
```
Ctrl + O, Ctrl + X
```bash
systemctl restart cron
```
Por fim, basta seguir a instalação do glpi. Ps : se for um glpi já existente, também é
possível realizar a migração dos dados para que fiquem fora da pasta do glpi


### Informação:

**/var/www/glpi** : diretório da pasta raiz do glpi

**/etc/glpi** : diretório para os arquivos do banco

**/var/lib/glpi** : diretório que substitui a pasta 'files' interna do GLPI

**/var/log/glpi** : diretório para os logs do glpi

É isso
Referência : https://glpi-install.readthedocs.io/pt/latest/install/index.html


## 2 Criar VIrtualhost para acesso seguro ao GLPI
Desde a versão 10.0.7 do GLPI foi incluso uma subpasta nova ‘public’ onde na configuração do
seu virtualhost deves realizar um redirecionamento para um arquivo index.php que há dentro dela. Seguindo a documentação, segue arquivo :

1. acessar a guia dos vhosts do apache
```bash
cd /etc/apache2/sites-available/
```
2. Criar vhost
```bash
nano glpi.conf
```
3. setar este conteúdo :
```bash
<VirtualHost *:80>
ServerName glpi.labudemy2024.com

DocumentRoot /var/www/glpi/public

# If you want to place GLPI in a subfolder of your site (e.g. your virtual host is serving
#multiple applications),
# you can use an Alias directive. If you do this, the DocumentRoot directive MUST NOT target
#the GLPI directory itself.
# Alias "/glpi" "/var/www/glpi/public"

<Directory /var/www/glpi/public>
Require all granted

RewriteEngine On

# Ensure authorization headers are passed to PHP.
# Some Apache configurations may filter them and break usage of API, CalDAV, ...
RewriteCond %{HTTP:Authorization} ^(.+)$
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

# Redirect all requests to GLPI router, unless file exists.
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^(.*)$ index.php [QSA,L]
</Directory>
</VirtualHost>
```
4. Habilitar módulo rewrite
```bash
a2enmod rewrite
```
5. Ativar novo vhost e reiniciar o apache2
```bash
a2ensite glpi.conf
```
```bash
systemctl restart apache2
```
6. Para acessar o GLPI a partir do dns definido ‘glpi.labudemy2024.com’, podemos setar essa
informação no arquivos hosts do Windows
Basta acessar esse caminho no seu computador : C:\Windows\System32\drivers\etc
Copie o arquivo ‘hosts’ para outro local fora dessa pasta e abra

Dentro do arquivo sete o dns dessa forma >> “ip” “dns’
Salve o arquivo, copie novamente para o caminho original e substitua-o
Por fim, já conseguiremos acessar o glpi no navegador utilizando o dns
glpi.labudemy2024.com

Os : Caso fosse realizado uma configuração do GLPi para acesso externo, seguiria a mesma
lógica. A diferença é que ao invés do ip interno seria o externo e ao invés de fazermos isso no
host da máquina, nós faríamos no gerenciador de dns(cloudlare, registro br etc)


## 3 Para utilizar os comandos do console do glpi precisamos garantir o bom funcionamento dos utilitários : composer e npm

### Instalar composer:
```bash
sudo apt update
```
```bash
sudo apt install curl php-cli php-mbstring git unzip
```
```bash
cd ~
```
```bash
curl -sS https://getcomposer.org/installer -o composer-setup.php
```
```bash
HASH=`curl -sS https://composer.github.io/installer.sig`
```
```bash
echo $HASH
```
```bash
php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; }
else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
```
```bash
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```
Por fim :
```bash
 composer
```

Referencia : https://www.digitalocean.com/community/tutorials/how-to-install-and-use-
composer-on-debian-11

### Instalar NPM
```bash
cd ~
```
```bash
curl -sL https://deb.nodesource.com/setup_18.x | bash -
```
```bash
apt install nodejs
```
```bash
node -v
```
```bash
apt install npm
```
```bash
npm -v
```
Instalar phpgettext(resolve o erro ao compilar as traduções/locales)
```bash
apt install gettext
```
### Acessar a URL gpli.set2024.com
 Rodar o comando 
 ```bash
/var/www/glpi/php <comando que aparece no navegador>
Atualiza a página
/var/www/glpi/php <comando que aparece no navegador>
```
### Agora vai entrar no modo assistente de instalação do GLPI.
Para resolver o erro da pasta marketingplace rode o comando
 ```bash
chown www-data:www-data /var/www/glpi/marketplace -Rf
```
### Inserir as credenciais do banco e concluir a instalação

Criar o usuário user.user como super admin e trocar a senha dos usuários padrão.

Excluir a pasta /install

## 4 Ajuste necessário para garantir a integridade dos horários no chamados tanto no GLPI quanto no banco de dados

Comandos necessários :

### Passo 1 : setar fuso horário do servidor
Ver lista de timezones disponíveis : 
```bash
timedatectl list-timezones
```
```bash
timedatectl set-timezone America/Sao_Paulo
```

Validar mudança : 

```bash
timedatectl
```





### Passo 2: compartilhar fuso horários com o mysql
```bash
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -p -u root mysql
```
“senha de root”
```bash
mysql -p
```
“senha de root”
```bash
SET GLOBAL time_zone = 'America/Sao_Paulo';
```
```bash
GRANT SELECT ON `mysql`.`time_zone_name` TO 'connectdb'@'localhost';
```
```bash
Quit;
```
```bash
systemctl restart mariadb.service
```
### Passo 3 : Ativar o uso do fuso horário no GLPI
Acessar /var/www/glpi
```bash
cd /var/www/glpi
```
Rodar o comando : 
```bash
php bin/console database:enable_timezones
```

Depois acessar no GLPI : Configurar > Geral > Valores Padrão
Na opção ‘Fuso horário’ setar o mesmo fuso horário escolhido no servidor

**Pronto, agora terás a informação do horário Ok no glpi, banco e servidor**
