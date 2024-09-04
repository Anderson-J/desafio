# Desafio

![Capa](./img/1.png)

## Índice

1. [Introdução](#introdução)
2. [Especificações](#especificações)
3. [Topologia simplificada](#topologia-simplificada)
4. [Pré requisitos](#pré-requisitos)
5. [Procedimento técnico](#procedimento-técnico)
    5. 1. [Primeira etapa - OpenCMS](#primeira-etapa---opencms)
    5. 2. [Segunda etapa - PostgreSQL](#segunda-etapa---postgresql)
    5. 3. [Terceira etapa - instalação do OpenCMS](#terceira-etapa---instalação-do-opencms)
    5. 4. [Quarta etapa - Nginx](#quarta-etapa---nginx)
6. [Referências](#referencias)

---

## Introdução

Essa documentação tem por objetivo documentar a entrega do teste técnico solicitado, assim como tornar transparente a implementação do OpenCMS e a utilização do Nginx como proxy reverso a fim de contemplar o encaminhamento de acessos recebidos para a aplicação, farei uma apresentação guiada, assim como evidenciarei artefatos do procedimento de implementação do OpenCMS.


## Especificações


O objeto de entrega é o acesso ao OpenCMS através do NGINX como proxy reverso. Onde:

- Deve existir a instalação e configuração do OpenCMS utilizando o PostgreSQL como banco de dados.

- O sistema operacional do servidor (uma máquina virtual, ou VM) deve ser Linux.

- Deve existir o acesso a aplicação através de um proxy reverso Nginx

## Topologia simplificada


![topologia](./img/2.png)

- Nginx – Proxy Reverso:  
    Hostname: debian-proxy  
    IPV4: 172.20.0.88/24  
    DNS: proxy.local  
- Nginx – Proxy Reverso:  
    Hostname: debian-proxy  
    IPV4: 172.20.0.88/24  
    DNS: proxy.local  


- App Server:  
    Hostname: debian-app  
    IPV4: 172.20.0.89/24  
    DNS: opencms.local  
- App Server:  
    Hostname: debian-app  
    IPV4: 172.20.0.89/24  
    DNS: opencms.local  


- Data Base Server:  
    Hostname: debian-db  
    IPV4: 172.20.0.74/24  
    DNS: database.local  
- Data Base Server:  
    Hostname: debian-db  
    IPV4: 172.20.0.74/24  
    DNS: database.local  

A escolha dessa arquitetura foi devido ao fato da facilidade de manutenções futuras, troubleshooting em caso de necessidade de inspeção de erros e tracing de logs, tendo cada recurso isolado em uma VM própria para cada serviço podemos ser mais assertivos em atualizações futuras além de diminuir o acoplamento de recursos em um único servidor.


## Pré requisitos

- 3 VMs Linux 
- Java JDK, versão 5 ou superior
- Tomcat
- PostgreSQL
- OpenCMS
- Nginx


## Procedimento técnico

Iniciaremos o procedimento técnico com a instalação do servidor de aplicação OpenCMS, para isso considerarei que já provisionamos uma máquina Debian que será usada como base para a instalação das outras VMs, as especificações escolhidas para cobrir esse caso são:

- VM Template
    Debian 11  
    4 Vcpus  
    4 GB ram  
    50 GB disk  

```bash
cat /etc/os-release
```
![os-release](./img/3.png)

```bash
free -h
```
![free -h](./img/4.png)

```bash
df -h --total
```
![df -h --total](./img/25.png)

```bash
lscpu | head -n 10
```
![lscpu](./img/5.png)

Considere também que o sistema estará com as últimas atualizações instaladas e para garantir isso faremos a atualização do sistema com o comando:
```bash
sudo apt update && sudo apt upgrade -y
```

Também faremos a instalação do editor de textos VIM com o comando:
```bash
sudo apt-get install vim
```

E editaremos o arquivo “/etc/hosts” usando a ferramenda VIM, de modo que a configuração final fique semelhante a essa em todas as VMs:

![cat /etc/hosts](./img/6.png)

Segue abaixo o teste de comunicação entre as VMs:

![ping -c3](./img/7.png)


### Primeira etapa - OpenCMS

Uma vez que a comunicação entre as máquinas está confirmada, podemos dar sequencia ao deploy do OpenCMS e para isso faremos a instalação dos componentes pré requisitos para o correto funcionamento da aplicação, são eles:

- Java JDK
    Procuramos qual versão estava disponível no repositório padrão do Debian utilizando o comando:

    ```bash
    sudo apt-cache search openjdk | grep jdk 
    ```

    A versão 17 nos pareceu a melhor escolha por ser a mais recente e estável disponível no repositório e a instalamos com o seguinte comando:

    ```bash
    sudo apt-get install openjdk-17-jdk -y
    ```

    ![java --version](./img/8.png)

- Tomcat9

    A escolha do Tomcat seguiu a mesma linha de raciocínio do Java, procurei no repositório padrão, com o comando:
    
    ```bash
    sudo apt-cache search tomcat
    ```


    Optamos por instalar versão estável disponível e realizamos a instalação com o comando:

    ```bash
    sudo apt-get install tomcat9 -y
    ```

    ```bash
    sudo /usr/share/tomcat9/bin/version.sh
    ```

    ![tomcat version](./img/9.png)

    Caso a instalação tenha sido bem sucedida será possível acessar o caminho especificado anteriormente na porta padrão de escuta do Tomcat conforme imagem abaixo:

    ![it works!](./img/10.png)



Para a aquisição da aplicação criamos uma pasta chamada /root/opencms com o comando:

```bash
mkdir /root/opencms && cd /root/opencms
```

e utilizamos a ferramenta wget para baixar a versão atual do OpenCMS diretamente do site ofical para a pasta citada:

```bash
wget http://www.opencms.org/downloads/opencms/opencms-17.0.zip
```

Neste ponto precisaremos de algumas ferramentas adicionais que não vem instaladas por padrão no Debian, são elas a “zip e unzip” o comando de instalação das mesmas é o que se segue:

```bash
sudo apt-get install zip unzip -y
```

e utilizar o comando unzip para descompactar o arquivo baixado:

```bash
unzip opencms-17.0.zip
```

entre os arquivos descompactados existe um arquivo chamado “opencms.war” esse arquivo deve ser movido para a pasta padrão de sites do servidor web Tomcat que no caso é a ***“/var/lib/tomcat9/webapps”***, a partir desse ponto seguiremos com a instalação através do painel web disponibilizado pela aplicação que pode ser acessado no endereço padrão do Tomcat + endereço da aplicação sendo assim:

> http://opencms.local:8080/opencms/setup

![welcome to alkacon](./img/11.png)

Após o aceite dos temos, nos depararemos com a seguinte tela:

![component tests](./img/12.png)

E se for bem sucedido na etapa anterior, seguiremos para a etapa de conexão com o banco de dados com a seguinte tela:

![database](./img/13.png)

Vale lembrar que usaremos um banco de dados PostgreSQL, portanto caso outra opção de banco esteja marcada troque para a opção correta.

Sendo assim, aqui encerra-se a primeira etapa para a instalação do OpenCMS. Para a segunda etapa será necessário a instalação do banco de dados, portanto a partir desse ponto daremos seguimento no provisionamento da VM do banco de dados.


### Segunda etapa - PostgreSQL

Tomando o sistema Debian padrão como base e para simplificar a adoção do banco de dados PostgreSQL, utilizamos a mesma linha de raciocínio para instalação do PostgreSQL, pesquisamos o repositório padrão e nos decidimos pela versão com suporte para casos de intervenções futuras, seguem os comando utilizados abaixo:

Comando para buscar e filtrar os resultados de PostgreSQL:

```bash
apt-cache search postgresql | grep postgre | head -n 15”
```

Comando para instalar o PostgreSQL e seus adicionais:

```bash
sudo apt-get install postgresql postgresql-contrib -y
```

Após a instalação utilizaremos a conta ***“postgres”*** que já existe e possui permissões de criação de banco de dados no servidor, assim, iremos alterar a senha para nossa conveniência. Esse usuário será utilizado na etapa de conexão com o banco de dados na interface web do OpenCMS. O procedimento segue os passos a seguir:

eleve as permissões de usuário ao se tornar root com o comando:

```bash
sudo -i
```
Entre no usuário ***"postgres"***:

```bash
su - postgres
```

Após isso entre na interface de linha de comando do PostgreSQL

```bash
psql
```

Altere a senha do ususário “postgres” para “senha” com o comando:

```bash
ALTER USER postgres WITH PASSWORD 'senha';
```

Saia da linha de comando e retorne ao usuário root:

```bash
\q
```

```bash
exit
```

Para que exista comunicação entre o Debian onde está instalado o OpenCMS e a outra VM onde está o Banco de dados para com o SGBD é necessário alterar alguns arquivos de configuração para permitir o acesso através de outro host, segue o passo a passo de como realizar essas alterações:

Como root entre na seguinte pasta com o comando:

```bash
cd /etc/postgresql/13/main/
```

Edite o arquivo ***"postgresql.conf"***:

```bash
vim postgresql.conf
```

Encontre a linha “listen_addresses” e modifique para adicionar a porta de escuta no host desejado, confome a imagem abaixo:

![listen_addresses](./img/14.png)

Também é necessário alterar as permissões de acesso a host externo no arquivo  ***“pg_hba.conf”*** que se encontra na mesma pasta e adicione essa sentença no arquivo:

> host    all             all             opencms.local             md5

salve os arquivos e reinicie o serviço do postgresql com o comando:

```bash
systemctl restart postgresql
```


após isso confirme que o serviço está sendo escutado na porta correta com o comando

```bash
ss -nltp
```

![ss-nltp](./img/15.png)

Finalizada a instalação do Banco de dados, assim como dos requisitos para a instalação do OpenCMS, voltaremos a interface web para dar seguimento na instalação da aplicação.


### Terceira etapa - instalação do OpenCMS

Utilize as credenciais “postgres” e “senha” conforme modificamos em algumas etapas acima e em seguida defina um usuário que será criado junto com o banco de dados, no nosso caso escolhemos o usuário “opencmsuser” e “senha” como credenciais.

Também realizaremos o apontamento para o servidor de banco de dados utilizando:

> jdbc:postgresql://database.local:5432/

![jdbc](./img/16.png)

Para nossa conveniência seguiremos com a configuração padrão:

![setup](./img/17.png)

![server settings](./img/18.png)

![importing modules](./img/19.png)

![finished](./img/20.png)

![congratulations](./img/21.png)

A instalação foi concluída com sucesso, resta agora a configuração do servidor de proxy reverso Nginx.


### Quarta etapa - Nginx

Tomaremos como base a VM Debian para instalar o Nginx da mesma forma que foi com os outros requisitos, utilizamos o seguinte comando:

```bash
sudo apt install nginx -y
```

como root, após a instalação é possível checar a versão com o comando:

```bash
nginx -v
```

![nginx -v](./img/22.png)

Assim como saber se o nginx está ativo utilizando o comando:

```bash
systemctl status nginx
```

Vale ressaltar que esse comando é util para saber o estado de qualquer serviço rodando nos servidores, esse comando não é exclusivo do nginx.

Para iniciar a configuração do nginx criaremos um arquivo de configuração dentro do caminho ***“/etc/nginx/sites-enabled/”*** com o comando:

```bash
vim /etc/nginx/sites-enabled/opencms
```

o arquivo de configuração deve estar da seguinte maneira:

```nginx
“server {
    listen 80;
    server_name proxy.local;

    location /opencms/ {
        proxy_pass http://opencms.local:8080/opencms/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
 
   include proxy_params;
}”
```

Após salvar o arquivo utilize o comando a seguir para reacarregar as configurações do nginx:

```bash
nginx -s reload
```

![nginx conf](./img/23.png)

Dessa maneira quando o endereço “http://proxy.local/opencms” for acessado seremos imediatamente direcionados para a página correspondente da aplicação:

![aplicacao](./img/24.png)


## Referências

[Documentação OpenCMS](https://documentation.opencms.org/central/)  
[Documentação PostgreSQL](https://www.postgresql.org/docs/)  
[Documentação Tomcat](https://tomcat.apache.org/tomcat-8.5-doc/index.html)  
[Documentação OpenJDK 17](https://devdocs.io/openjdk~17/)  
[Documentação Nginx](https://nginx.org/en/docs/)  
