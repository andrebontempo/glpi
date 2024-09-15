
##  Atualização do GLPI

### 1. Local para salvar os backups ? > sugestão : pasta raiz do usuário root ou o usuário
utilizado na instalação
```bash
cd /root
```
### 2. Realizar backup do banco de dados(exportação)
```bash
mysqldump -u usuario -p nome_do_banco > nome_do_arquivo.sql
```
Será salvo no diretório atual

### 3. Parar serviço do apache(para evitar erros)
```bash
systemctl stop apache2
```
### 4. Renomear pasta do glpi atual para _old
```bash
cd /var/www/
```
```bash
mv glpi glpi_old

```
### 5. Baixar, descompactar e renomear pasta nova do GLPI
```bash
wget https://github.com/glpi-project/glpi/archive/refs/tags/10.0.11.zip
```
```bash
unzip 10.0.11.zip
```
```bash
mv glpi-10.0.11 glpi
```
### 6. Copiar conteúdo necessário da pasta antiga
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
### ***Fim***
