# Nginx Server Blocks (Virtual Host) - Linux

Um host virtual é um termo do Apache, entretanto, é comumente usado por usuários do Nginx também. O termo apropriado para Nginx é bloco de servidor (Server Blocks). Ambas as palavras têm o mesmo significado, que é basicamente a característica de poder hospedar vários sites em um único servidor. Isso é extremamente útil, visto que você possui vários sites e não deseja passar pelo longo (e caro) processo de configuração de um novo servidor web para cada site.

#### Pré-Requisitos:
 - Servidor Ubuntu configurado (root/usuário)
 - Nginx instalado
```sh
$sudo apt-get update
$sudo apt-get install nginx
```
Assim que tiver realizado todos os requerimentos, pode seguir os próximos passos.

Em todos os passos estarei utilizando um usuário não-root, por isso possui `sudo` antes de todos os comandos. Lembrando que caso siga todos os passos como root, não necessita utilizar `sudo`.

Para demonstração vou configurar 2 domínios dentro do nosso servidor Nginx, `teste1.com` e `teste2.com`.

### 1° Passo - Criação de diretórios
Por padrão o Nginx utiliza o seguinte diretório:
```sh
/usr/share/nginx/html
```
Porém para ficar mais fácil e por padronização de Hosts Virtuais, utilizaremos o seguinte caminho:
```sh
/var/www
```
Então verifique a existencia deste diretório. Caso não exista, vamos criá-lo já introduzindo o caminho do nossos Hosts `teste1.com` e `teste2.com`.
```sh
$sudo mkdir -p /var/www/teste1.com/html
$sudo mkdir -p /var/www/teste2.com/html
```
Agora com os diretórios criados, vamos dar permissão ao nosso usuário logado. Para que possa criar arquivos. (vamos supor que o usuário logado se chame "joao").
```sh
$sudo chown -R joao:joao /var/www/teste1.com/html
$sudo chown -R joao:joao /var/www/teste2.com/html
$sudo chmod -R 755 /var/www
```
### 2° Passo - Criação das páginas

Agora vamos criar as páginas que são hospedadas.
```sh
$sudo nano /var/www/teste1.com/html/index.html
```
E criaremos uma página simples, apenas para demonstração!
```html
<html>
    <head>
        <title>Teste 1</title>
    </head>
    <body>
        <h1>Sucesso</h1>
        <p>A nossa primeira página está no ar!</p>
    </body>
</html>
```
E salve o arquivo.

Para facilitar a criação da segunda página.
```sh
$cp /var/www/teste1.com/html/index.html /var/www/teste2.com/html/
```
E altere apenas alguns detalhes.
```html
<html>
    <head>
        <title>Teste 2</title>
    </head>
    <body>
        <h1>Sucesso</h1>
        <p>A nossa segunda página está no ar!</p>
    </body>
</html>
```
Feche-o se salve o segundo arquivo.
### 3° Passo - Criação do primeiro Bloco do Servidor (Server Block)
Agora vamos começar com a configuração dos blocos. Como padrão, já existe um arquivo, nomeado de `default` dentro de `/sites-available.` Para facilitar, copie-o com o domínio de seu primeiro Host.

```sh
$sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/teste1.com
$sudo nano /etc/nginx/sites-available/teste1.com
```

Agora vamos configura-lo.

```sh
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /usr/share/nginx/html;
    index index.html index.htm;

    server_name localhost;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
Este será o arquivo que encontrará (desconsiderando as linhas comentadas).

A primeira coisa que devemos fazer é direcionar o root, apontando para o site.
```sh
root /var/www/teste1.com/html;
```
Em seguida vamos alterar o `server_name` o que irá fazer a requisição do domínio
```sh
server_name teste1.com www.teste1.com;
```

### 4° Passo - Criação do segundo Bloco do Servidor (Server Block)
Seguimos para a configuração do segundo bloco de servidor. Para facilitar, vamos copiar o primeiro bloco e apenas fazer as devidas alteraçõs

```sh
$sudo cp /etc/nginx/sites-available/teste1.com /etc/nginx/sites-available/test2.com
$sudo nano /etc/nginx/sites-available/teste2.com
```
Como o `teste1.com` já está configurado como `default_serever`, devemos retirar esta informação no `teste2.com` para que não entre em conflito, alterando a porta da seguinte forma:
```sh
    listen 80;
    listen [::]:80;
```
E como feito anteriormente, trocaremos o `root` e o `server_name`
```sh
root /var/www/teste2.com/html;
server_name teste2.com www.teste2.com;
```

### 5° Passo - Habilitando os Blocos do Servidor (Server Block)
Para habilitarmos, vamos utilizar a técnica de criar Links Simbólicos dentro do diretório `/sites-enabled`, o diretório que o Nginx faz a leitura assim que iniciado
```sh
$sudo ln -s /etc/nginx/sites-available/teste1.com /etc/nginx/sites-enabled/
$sudo ln -s /etc/nginx/sites-available/teste2.com /etc/nginx/sites-enabled/
```
Agora eles estão no diretório `/sites-enabled`. Mas ainda existe um último problema, o arquivo template `default` que havíamos usado anteriormente ainda está habilitado, então devemos desabilitar, para conseguirmos prosseguir.
```sh
$sudo rm /etc/nginx/sites-enabled/default
```
### 6° Passo - ùltimas configurações NGINX
Vamos fazer a última alteração na .conf para que funcione perfeitamente.
Abra o arquivo:
```sh
$sudo nano /etc/nginx/nginx.conf
```
Descomente a seguinte linha:
```sh
server_names_hash_bucket_size 64;
```
Salve a alteração e inicie novamente o NGINX para aplica-las
```sh
$sudo service nginx restart
```
### 7° Passo - ùltimas configurações NGINX
Agora vamos configurar para conseguirmos acessor o nosso site, através domínio configurado.

Isso não irá permitir que outros visualizem seu site, mas permitirá que você alcance cada site independentemente e assim testando sua configuração. Basicamente, isso funciona interceptando solicitações que normalmente vão para o DNS para resolver nomes de domínio. Em vez disso, podemos definir os endereços IP que queremos que nosso computador local vá quando solicitamos os nomes de domínio.
```sh
$sudo nano /etc/hosts
```
Você precisa do endereço IP público do seu servidor e dos domínios que deseja rotear para o servidor. Para obter ele basta utilizar o seguinte comando
```sh
host myip.opendns.com resolver1.opendns.com | grep "myip.opendns.com has" | awk '{print $4}'
```
```sh
127.0.0.1   localhost
xxx.xxx.xxx.xxx teste1.com
xxx.xxx.xxx.xxx teste2.com
```
ou
```sh
127.0.0.1   localhost
127.0.0.1   teste1.com
127.0.0.1   teste2.com
```
Agora salve o arquivo.
# Vamos ver o Resultado
Agora que terminando de configurar tudo, vamos testar se os blocos de servidor estão funcionando corretamente. Você pode fazer isso visitando os domínios em seu navegador da web:
```
http://teste1.com
```
e depois
```
http://teste2.com
```
Agora podemos ver que ambos os domínios estão funcionando corretamente, significa que você configurou corretamente 2 blocos de servidores (Server Blocks) independentes dentro do NGINX.

Agora para acrescentar novos domínios, apenas seguir os passos 4 ao 7 novamente e estará tudo funcionando 100%