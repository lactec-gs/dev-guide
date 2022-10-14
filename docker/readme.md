
# Instalação o Docker no Ubuntu

Certifique-se de não estar conectado na VPN do Lactec, pois parece haver um
bloqueio no download dos pacotes.

## Conteúdo

* [Instalação do Ubuntu no Windows](#A)  
* [Instalação do Docker Engine](#1)  
* [Instalação do Docker Compose](#2)
* [Instalação do Portainer](#3)
* [Verificação do Docker no Windows](#4)
* [Observações](#5)
* [Configurar o direcionamento de porta para acesso remoto](#6)
* [Iniciar Docker automaticamente ao iniciar o Windows](#7)
* [Espaço em Disco](#espaco)
* [Fontes](#8)

<a id="A"></a>

## Instalação do Ubuntu no Windows

Ver todos os passos detalhados no site <https://betterprogramming.pub/how-to-install-docker-without-docker-desktop-on-windows-a2bbb65638a1>

Abrir o PowerShell como administrador e execute os comandos a seguir:

1. `Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux`

2. `wsl --set-default-version 2`  
   `wsl --install -d Ubuntu`

3. `wsl -l -v`

<a id="1"></a>

## Instalação do Docker Engine

1. Executar o Ubuntu;

2. Execute o conteúdo a seguir em sequência:

    ```text
    # 1. Required dependencies
    sudo apt-get update
    sudo apt-get -y install apt-transport-https ca-certificates curl gnupg lsb-release

    # 2. GPG key
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

    # 3. Use stable repository for Docker
    echo \
   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

    # 4. Install Docker
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

    # 5. Add user to docker group
    sudo groupadd docker
    sudo usermod -aG docker $USER
    ```

3. Se houver erro na instalação via script, tente executar as etapas no site: <https://docs.docker.com/engine/install/ubuntu>

4. Iniciar o docker:  

    ```text
    sudo service docker start   # Inicar o serviço do Docker
    sudo service docker status  # Verificar o status do serviço
    ```

    **Nota 1**:  
    É necessário executar o comando `sudo service docker start` no terminal
    do Ubuntu toda vez que a máquina é reiniciada para iniciar o serviço Docker.

    **Nota 2**:  
    Outra forma de iniciar o serviço Docker é através do comando abaixo, a ser
    executado no GitBash do Windows:  
    `wsl -u root service docker status > /dev/null || wsl -u root service docker start > /dev/null`  

    ou criar um script chamado `iniciar_docker.sh` em algum local de interesse com
    o script:

    ```text
    wsl -u root service docker status > /dev/null || wsl -u root service docker start > /dev/null
    ```

    Para executar o script no GitBash: `source iniciar_docker.sh`

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

5. Veja o item [Espaço em Disco](#espaco) após a instalação

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

2. Se ao digitar `docker` no terminal e não reconhecer o comando, é porque o não
 foi instalado.

3. Recomendo tentar a instalação a partir do url <https://docs.docker.com/engine/install/ubuntu/>

4. Se aparecer mensagem de permissão negada, realizar os passos abaixo:

    ```text
    sudo groupadd docker
    sudo usermod -aG docker $USER
    newgrp docker
    ```

<a id="6"></a>

## Configurar o direcionamento de porta para acesso remoto

Se preferir automatizar esta etapa, vá para item [Iniciar Docker automaticamente ao iniciar o Windows](#7)
e pule esta etapa.

Esta etapa é opcional, mas será necessário realizar caso você deseja conectar a
um container Docker a partir de outra máquina. Será necessário realizar os
comandos abaixo para cada porta.

Executar o PowerShell como administrador:

1. Verificar os redirecionamentos existentes: `netsh interface portproxy show v4tov4`

2. Adicionar o redirecionamento da porta: `netsh interface portproxy add v4tov4 listenport=9000 listenaddress=192.168.1.8 connectport=9000 connectaddress=$($(wsl hostname -I).Trim().Split(' ')[0]);`  

No exemplo, 9000 é a porta para redirecionar. 192.168.1.8 é o IP da máquina Windows (host)

3. Para deletar um redirecionamento: `netsh interface portproxy delete v4tov4 listenport=9000 listenaddress=192.168.1.8`

<a id="7"></a>

## Iniciar Docker automaticamente ao iniciar o Windows

1. Crie um script chamado ***wsl_iniciar_docker.ps1*** num diretório qualquer e copie o
    conteúdo abaixo:
    Se quiser mapear outras portas, só adicionar a linha:

    `netsh interface portproxy add v4tov4 listenport=PORTA listenaddress=$ip_machine connectport=PORTA connectaddress=$($(wsl hostname -I).Trim().Split(' ')[0]);`

    No exemplo abaixo, está mapeada apenas a porta 9000.

    ```text
    # Iniciar o serviço Docker na Distro
    wsl -u root service docker start

    # Obter o IP da máquina (Windows)
    $ip_machine = (Get-NetIPConfiguration | Where-Object {$_.IPv4DefaultGateway -ne $null -and $_.NetAdapter.Status -ne "Disconnected"}).IPv4Address.IPAddress

    # Mapear o host e porta entre a distribuição e o Windows
    netsh interface portproxy add v4tov4 listenport=9000 listenaddress=$ip_machine connectport=9000 connectaddress=$($(wsl hostname -I).Trim().Split(' ')[0]);
    ```

2. Para executar o script, é necessário desativar uma configuração de segurança do
    Windows. Abra o PowerShell como administrador e execute os comando abaixo.
    Este comando é necessário apenas uma vez.  
    `Set-ExecutionPolicy RemoteSigned`  

3. Para executar o script, digite o comando abaixo no PowerShell como administrador.  
`.\wsl_iniciar_docker.ps1`

4. Para o Docker iniciar automaticamente com o Windows, será necessário criar uma
 tarefa no agendador de tarefas do Windows. Para facilitar, use como referência
 o conteúdo abaixo. Crie um arquivo com qualquer nome com extensão .xml, copie o
 e modifique conforme suas necessidades.  
 Após salvar o arquivo, basta importar no Agendador de Tarefas do Windows. Será
 necessário alterar o usuário e caminho do script.ps1

    ```text
    <?xml version="1.0" encoding="UTF-16"?>
    <Task version="1.4" xmlns="http://schemas.microsoft.com/windows/2004/02/mit/task">
    <RegistrationInfo>
        <Date>2022-08-25T16:35:28.0523306</Date>
        <Author>LACTEC\L01647</Author>
        <Description>Observação: a máquina deve ter somente uma distro instalada</Description>
        <URI>\Iniciar Docker na Distro Linux</URI>
    </RegistrationInfo>
    <Triggers>
        <LogonTrigger>
        <Enabled>true</Enabled>
        <UserId>LACTEC\L01647</UserId>
        </LogonTrigger>
    </Triggers>
    <Principals>
        <Principal id="Author">
        <UserId>S-1-5-21-2775306953-3624129364-2707181804-30689</UserId>
        <LogonType>S4U</LogonType>
        <RunLevel>HighestAvailable</RunLevel>
        </Principal>
    </Principals>
    <Settings>
        <MultipleInstancesPolicy>IgnoreNew</MultipleInstancesPolicy>
        <DisallowStartIfOnBatteries>true</DisallowStartIfOnBatteries>
        <StopIfGoingOnBatteries>true</StopIfGoingOnBatteries>
        <AllowHardTerminate>true</AllowHardTerminate>
        <StartWhenAvailable>false</StartWhenAvailable>
        <RunOnlyIfNetworkAvailable>false</RunOnlyIfNetworkAvailable>
        <IdleSettings>
        <StopOnIdleEnd>true</StopOnIdleEnd>
        <RestartOnIdle>false</RestartOnIdle>
        </IdleSettings>
        <AllowStartOnDemand>true</AllowStartOnDemand>
        <Enabled>true</Enabled>
        <Hidden>true</Hidden>
        <RunOnlyIfIdle>false</RunOnlyIfIdle>
        <DisallowStartOnRemoteAppSession>false</DisallowStartOnRemoteAppSession>
        <UseUnifiedSchedulingEngine>true</UseUnifiedSchedulingEngine>
        <WakeToRun>false</WakeToRun>
        <ExecutionTimeLimit>PT0S</ExecutionTimeLimit>
        <Priority>7</Priority>
    </Settings>
    <Actions Context="Author">
        <Exec>
        <Command>C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe</Command>
        <Arguments>"C:\_Dev\Docker\wsl_iniciar_docker.ps1"</Arguments>
        </Exec>
    </Actions>
    </Task>
    ```

Outros comandos úteis:

* Executar uma distribuição wsl: `wsl -d Ubuntu`  

* Listar todas as distribuições em execução: `wsl --list --running`

* Desligar as distribuições: `wsl --shutdown`

<a id="espaco"></a>

## Espaço em Disco

Um problema conhecido ao usar o Docker em WSL2 no Windows é o tamanho do arquivo
da imagem "ext4.vhdx", salvo no diretório parecido como "C:\Users\NOME_DO_USUARIO\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState"

Por mais que imagens não utilizadas no Docker são deletadas, o tamanho do arquivo
não diminui e cresce conforme o uso.
Ver mais aqui <https://github.com/microsoft/WSL/issues/4699>

Existem algumas soluções para compactar da imagem, mas o mesmo não faz milagre.

1. Se quiser tentar compactar, segue as etapas:

```text
wsl.exe --shutdown
cd C:\Users\NOME_DO_USUARIO\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\
optimize-vhd -Path .\ext4.vhdx -Mode full
```

2. Caso não diminuir, recomendo a exclusão da distribuição Linux e reinstalar a
nova distribuição. O problema deste método é a reinstalação do Docker. Porém, é
possível realizar o backup de uma distribuição Linux num arquivo .tar:

3. Uma dica é criar um backup da imagem após a instalação do Docker na distribuição.

    Exportar
    ```text
    cd C:\Users\NOME_DO_USUARIO\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState
    wsl --export Ubuntu C:\Ubuntu.tar
    ```
    Importar
    Preciso encontrar como fazer ainda, depois eu documento aqui. 
    Num site encontrei algo como abaixo:

    ```text
    wsl --unregister docker-desktop-data
    wsl --unregister docker-desktop
    # Import backup of old WSL images (can be from an older version of Docker Desktop)
    # wsl.exe --import <DistributionName> <InstallLocation> <FileName>
    # any <InstallLocation> should work (was `%LOCALAPPDATA%/Docker/wsl/data` before)
    wsl --import docker-desktop-data C:\Users\windows-admin\AppData\Local\Docker\wsl\data C:\Users\windows-admin\DockerVHDXs\docker-desktop-data.tar 
    wsl --import docker-desktop C:\Users\windows-admin\AppData\Local\Docker\wsl\distro C:\Users\windows-admin\DockerVHDXs\docker-desktop.tar
    ```


<a id="8"></a>

## Fontes

Este tutorial foi baseado no seguintes sites:

* <https://betterprogramming.pub/how-to-install-docker-without-docker-desktop-on-windows-a2bbb65638a1>

* <https://docs.docker.com/engine/install/ubuntu/>

* <https://askubuntu.com/questions/1355633/how-to-start-a-specific-service-when-ubuntu-is-started-on-wsl2>

* <https://github.com/codeedu/wsl2-docker-quickstart#docker-engine-docker-nativo-diretamente-instalado-no-wsl2>

* <https://www.youtube.com/watch?v=ACjlvzw4bVE&ab_channel=MatheusBattisti-HoradeCodar>

* <https://gist.github.com/datocrats-org/6c2bea8907a98299ac26dab413f0e3d8>
