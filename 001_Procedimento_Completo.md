## 1 Preparar VM linux com pacote web e banco de dados

### Instalar a VM Debian 12 e atualizar com o comando abaixo:
```bash
   apt -y update && apt -y upgrade && apt autoremove && apt autoclean
```


### Pacotes para manipular arquivos:
```bash
   apt install -y xz-utils bzip2 unzip curl
```
## Instalar dependências no sistema
```bash
apt install -y apache2 libapache2-mod-php php-soap php-cas php php-{apcu,cli,common,curl,gd,imap,ldap,mysql,xmlrpc,xml,mbstring,bcmath,intl,zip,bz2}
```
#Instalar o Serviço MySQL
apt install -y mariadb-server
#Criar Banco de Dados no Mysql
mysql -u root -p
mysql> create database glpi10 character set utf8mb4 collate utf8mb4_bin;
mysql> create user connectdb@localhost identified by 'glpilab@udemy2024';
mysql> grant all privileges on glpi10.* to connectdb@localhost;
mysql> flush privileges;
mysql> quit;

#Setar ipfixo e ativar acesso via ssh
apt install openssh-server

#Habilitar login root no ssh : 
nano /etc/ssh/sshd_config
Definir : 'PermitRootLogin yes'
Salvar e fechar : Ctrl + O, Enter, Ctrl + X
reiniciar o serviço : 
systemctl restart ssh

#Setar ip fixo na máquina : nano /etc/network/interfaces
Ps : ajuste para a faixa de ip, máscara etc da sua rede

iface "interface" inet static
address 192.168.1.80
netmask 255.255.255.0
network 192.168.1.0
broadcast 192.168.1.255
gateway 192.168.1.1

Salvar e fechar : Ctrl + O, Enter, Ctrl + X

#Reiniciar a vm : 
systemctl reboot

Por fim, validar se o acesso ssh continua OK
















Instalação GLPI 10 by Lucas Levi – 2024
• Passo 1 : Acessar 
cd /var/www
1.1 Baixar glpi : 
wget https://github.com/glpi-project/glpi/archive/refs/tags/10.0.10.zip
Referência : https://github.com/glpi-project/glpi/releases/

1.2 Descompactar pasta : 
unzip 10.0.10.zip
1.3Renomear pasta : 
mv glpi-10.0.10 glpi

• Passo 2
2.1 Após descompactar a pasta, acessar 
cd /inc 
e criar o arquivo 
nano downstream.php 
e setar esse conteúdo :
<?php
define('GLPI_CONFIG_DIR', '/etc/glpi/');
if (file_exists(GLPI_CONFIG_DIR . '/local_define.php')) {
require_once GLPI_CONFIG_DIR . '/local_define.php';
}

2.2 Criar os diretórios a seguir :
mkdir /etc/glpi
mkdir /var/lib/glpi
mkdir /var/log/glpi

2.3 Criar sub diretórios da pasta files
cd /var/www/glpi/files
cp * -Rf /var/lib/glpi

2.4 Dar permissões do usuário apache nessas pastas
chown www-data:www-data /etc/glpi -Rf
chown www-data:www-data /var/lib/glpi -Rf
chown www-data:www-data /var/log/glpi -Rf
chown www-data:www-data /etc/www/glpi -Rf

2.5 Em 
cd /etc/glpi 
criar o arquivo 
nano local_define.php
 com o seguinte conteúdo :
<?php
define('GLPI_VAR_DIR', '/var/lib/glpi');
define('GLPI_LOG_DIR', '/var/log/glpi');

2.6 Habilitar diretiva ‘session.cookie_httponly’ no PHP
Caminho 1 : 
nano /etc/php/8.2/cli/php.ini
Ctrl + W, session.cookie_httponly, session.cookie_httponly = ON
Ctrl + O, Ctrl + X
Caminho 2 : 
nano /etc/php/8.2/apache2/php.ini
Ctrl + W, session.cookie_httponly, session.cookie_httponly = ON
Ctrl + O, Ctrl + X
systemctl restart apache2
2.7 Ativar cron do GLPI
nano /etc/crontab
adicionar a linha : * * * * * root php -f /var/www/glpi/front/cron.php
Ctrl + O, Ctrl + X
systemctl restart cron

Por fim, basta seguir a instalação do glpi. Ps : se for um glpi já existente, também é
possível realizar a migração dos dados para que fiquem fora da pasta do glpi

__
Informação :
/var/www/glpi : diretório da pasta raiz do glpi
/etc/glpi : diretório para os arquivos do banco
/var/lib/glpi : diretório que substitui a pasta 'files' interna do GLPI
/var/log/glpi : diretório para os logs do glpi

É isso
Referência : https://glpi-install.readthedocs.io/pt/latest/install/index.html
Criar VIrtualhost para acesso seguro ao GLPI
Desde a versão 10.0.7 do GLPI foi incluso uma subpasta nova ‘public’ onde na configuração do
seu virtualhost deves realizar um redirecionamento para um arquivo index.php que há dentro dela. Seguindo a documentação, segue arquivo :
1. acessar a guia dos vhosts do apache
cd /etc/apache2/sites-available/
2. Criar vhost
nano glpi.conf
3. setar este conteúdo :
<VirtualHost *:80>
ServerName glpi.labudemy2024.com

DocumentRoot /var/www/glpi/public

# If you want to place GLPI in a subfolder of your site (e.g. your virtual host is serving
multiple applications),
# you can use an Alias directive. If you do this, the DocumentRoot directive MUST NOT target
the GLPI directory itself.
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

4. Habilitar módulo rewrite
a2enmod rewrite
5. Ativar novo vhost e reiniciar o apache2
a2ensite glpi.conf
systemctl restart apache2

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
Para utilizar os comandos do console do glpi precisamos garantir o bom funcionamento dos utilitários : composer e npm
Instalar composer:
sudo apt update
sudo apt install curl php-cli php-mbstring git unzip
cd ~
curl -sS https://getcomposer.org/installer -o composer-setup.php
HASH=`curl -sS https://composer.github.io/installer.sig`
echo $HASH
*****
php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; }
else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer

Por fim : composer

Referencia : https://www.digitalocean.com/community/tutorials/how-to-install-and-use-
composer-on-debian-11

Instalar NPM
cd ~
curl -sL https://deb.nodesource.com/setup_16.x | bash -
apt install nodejs
node -v
apt install npm
npm -v

Instalar phpgettext(resolve o erro ao compilar as traduções/locales)
apt install gettext
Ajuste necessário para garantir a integridade dos horários no chamados tanto no GLPI quanto no banco de dados
Comandos necessários :

➢ Passo 1 : setar fuso horário do servidor
Ver lista de timezones disponíveis : 
timedatectl list-timezones
timedatectl set-timezone America/Fortaleza
Validar mudança : 
timedatectl

Passo 2: compartilhar fuso horários com o mysql
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -p -u root mysql
“senha de root”
mysql -p
“senha de root”
SET GLOBAL time_zone = 'America/Fortaleza';
GRANT SELECT ON `mysql`.`time_zone_name` TO 'connectdb'@'localhost';
Quit;
systemctl restart mariadb.service

Passo 3 : Ativar o uso do fuso horário no GLPI
Acessar /var/www/glpi
Rodar o comando : 
php bin/console database:enable_timezones
Depois acessar no GLPI : Configurar > Geral > Valores Padrão
Na opção ‘Fuso horário’ setar o mesmo fuso horário escolhido no servidor

Pronto, agora terás a informação do horário Ok no glpi, banco e servidor

Atualização do GLPI
1. Local para salvar os backups ? > sugestão : pasta raiz do usuário root ou o usuário
utilizado na instalação
cd /root

2. Realizar backup do banco de dados(exportação)
mysqldump -u usuario -p nome_do_banco > nome_do_arquivo.sql - será salvo no
diretório atual

3. Parar serviço do apache(para evitar erros)
systemctl stop apache2

4. Renomear pasta do glpi atual para _old
Cd /var/www/
mv glpi glpi_old
5. Baixar, descompactar e renomear pasta nova do GLPI
wget https://github.com/glpi-project/glpi/archive/refs/tags/10.0.11.zip
unzip 10.0.11.zip
mv glpi-10.0.11 glpi
6. Copiar conteúdo necessário da pasta antiga
No nosso caso copiaremos a pasta plugins, pics, marketplace e inc
Para realizar isso utilizaremos o utilitário ‘mc’
apt install mc
digite no terminal : mc + Enter
//Seguir a realização das cópias conforme mostrado em vídeo//
Por fim, realizar novamente comando para privilégios do usuário apache :
chown www-data:www-data /var/www/glpi -Rf
Após isso, reiniciar o apache : systemctl restart apache2
E seguir a atualização pelo link raiz do glpi(no navegador)

Pós atualização : remover pasta ‘install’ da pasta raiz
Rm -rf Install/
-- fim –














Script Backup Banco de Dados By Aristides Neto

Passo 1 : definir um local e criar 2 pastas(1 para o backup do banco e o a outra para os arquivos
de logs)

Sugestão : mkdir /backup_glpi
Dentro de /backup_glpi criar backups_bancos e logs_backups
mkdir backups_bancos
mkdir logs_backups

Passo 2 : Criar arquivo .sh com o script
nano scriptbackup.sh

conteúdo : acessar o link e copiar >>> https://aristides.dev/script-de-backup-de-banco-de-
dados-mysql/

Passo 3 : Editar as linhas de acordo com suas informações :
# Define usuario e senha do banco
USER='root'
PASS='root'
# Caminho do arquivo de log
LOG_DIR=/var/log/backup
# Diretorio onde serão salvos os backups
DIR_BK=/var/backups/database
# Lista dos bancos de dados que serão realizados o backup
DATABASES=(banco01 banco02)

Ctrl + O, Ctrl + X
Habilitar execução do arquivo : chmod +x scriptbackup.sh
Testar script : ./ scriptbackup.sh
Acessar a pasta do backup e verificar se funcionou

Passo 4 : Ativar rotina de backup na cron do Linux

nano /etc/crontab
30 22 * * * root /backup_glpi/scriptbackup.sh
Ctrl + O, Ctrl + X
systemctl restart cron

nesse exemplo o backup será realizado todos os dias às 22h30
Ajuste ou acrescente novos horários de acordo com sua necessidade

--fim --




