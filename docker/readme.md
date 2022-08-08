
# Instalação o Docker no Ubuntu

Certifique-se de não estar conectado na VPN do Lactec, pois parece haver um
bloqueio no download dos pacotes.

Este tutorial foi baseado no seguintes sites:

* <https://betterprogramming.pub/how-to-install-docker-without-docker-desktop-on-windows-a2bbb65638a1>

* <https://docs.docker.com/engine/install/ubuntu/>

## Conteúdo

* [Instalação do Docker Engine](#1)  
* [Instalação do Docker Compose](#2)
* [Instalação do Portainer](#3)
* [Verificação do Docker no Windows](#4)
* [Observações](#5)

<a id="1"></a>

## Instalação do Docker Engine

1. Executar o Ubuntu;

2. Criar um script com um nome qualquer, exemplo install_docker.sh: `nano install_docker.sh`
3. Copiar o conteúdo abaixo no arquivo e salvar:

    ```text
    # /bin/bash

    # 1. Required dependencies
    sudo apt-get update
    sudo apt-get -y install apt-transport-https ca-certificates curl gnupg lsb-release

    # 2. GPG key
    curl -fsSL <https://download.docker.com/linux/ubuntu/gpg> | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

    # 3. Use stable repository for Docker
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] <https://download.docker.com/linux/ubuntu> \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

    # 4. Install Docker
    sudo apt-get update
    sudo apt-get -y install docker-ce docker-ce-cli containerd.io

    # 5. Add user to docker group
    sudo groupadd docker
    sudo usermod -aG docker $USER
    ```

4. Executar o arquivo: `sudo ./install_docker.sh`

6. Se houver erro na instalação via script, tente executar as etapas no site: <https://docs.docker.com/engine/install/ubuntu>

5. Iniciar o docker:  

    ```text
    sudo service docker start   # Inicar o serviço do Docker
    sudo service docker status  # Verificar o status do serviço
    ```

    ```text
        Nota: É necessário executar o comando "sudo service docker start" a cada 
        reinicio da máquina, já que não foi configurado para iniciar o serviço Docker
        de forma automática.
    ```

<a id="2"></a>

## Instalação do Docker Compose

Abra um terminal no Ubuntu:

1. Download:

    ```text
    sudo curl -L https://github.com/docker/compose/releases/download/v2.9.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
    ```

2. Dar permissão:

    ```text
    sudo chmod +x /usr/local/bin/docker-compose
    ```

3. Verificar a instalado, digite no terminal:  `docker-compose --version`

<a id="3"></a>

## Instalação do Portainer

O Portainer é uma ferramenta GUI que permite gerenciar as imagens Docker. Mais
informações podem ser encontradas no site: <https://www.portainer.io>.

1. Copie o conteúdo abaixo em arquivo chamado `docker-compose.yml` em diretório
a sua escolha:

    ```text
    version: '3'

    services:
        portainer:
            image: portainer/portainer-ce:latest
            container_name: portainer
            restart: unless-stopped
            security_opt:
            - no-new-privileges:true
            volumes:
            - /etc/localtime:/etc/localtime:ro
            - /var/run/docker.sock:/var/run/docker.sock:ro
            - ./portainer-data:/data
            ports:
            - 9000:9000
    ```

2. Em seguida execute o seguinte comando: `wsl docker-compose up -d`

3. Abra no navegador o endereço <http://localhost:9000>.

4. Certifique-se de fazer login e criar suas credenciais logo após o Portainer
estar pronto, ou ele se desligará automaticamente por segurança. Se você não
criou as credenciais a tempo e ele foi encerrado automaticamente, será necessário
reiniciar o serviço.

<a id="4"></a>

## Verificação do Docker no Windows

Para verificação no Windows, abra um terminal. Por exemplo, no PowerShell,
Prompt de Comando ou Git Bash, digite os seguintes comandos:

* Docker: `wsl docker ps`
* Docker-Compose: `wsl docker-compose --version`

Dica:
É possível configurar um alias para os comandos no Windows. No Gitbash, editar
o arquivo ***aliases.sh*** localizado no diretório
***C:\Program Files\Git\etc\profile.d***

Adicionar os seguintes aliases:

```text
alias docker='wsl docker $args'
alias docker-compose='wsl docker-compose $args'
```

Após adicionar os aliases, reabra o GitBash e digite os comandos abaixo para
testar:

* Docker: `docker ps`
* Docker-Compose: `docker-compose --version`

<a id="5"></a>

## Observações

1. Verifique se o proxy não está ativo.

2. Se ao digitar `docker` no terminal e não reconhecer o comando, é porque o não foi instalado.

3. Recomendo tentar a instalação a partir do url <https://docs.docker.com/engine/install/ubuntu/>

4. Se aparecer mensagem de permissão negada, realizar os passos abaixo:

    ```text
    sudo groupadd docker
    sudo usermod -aG docker $USER
    newgrp docker
    ```
