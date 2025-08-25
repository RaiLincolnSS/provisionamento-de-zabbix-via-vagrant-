# Zabbix Multi-Tier com Vagrant

Este projeto utiliza o Vagrant para automatizar a criação de um ambiente de monitorização Zabbix completo, distribuído numa arquitetura de 3 camadas (3-tier). Cada camada (Banco de Dados, Servidor de Aplicação e Frontend Web) é provisionada numa máquina virtual separada, proporcionando um ambiente isolado e organizado, ideal para laboratórios, testes ou estudos.

## Arquitetura

O ambiente é composto por três servidores virtuais, todos a correr **Ubuntu 22.04 LTS (Jammy Jellyfish)**:

1.  **Servidor de Banco de Dados (`zabbix-db`)**
    * **IP:** `192.168.3.110`
    * **Software:** PostgreSQL
    * **Função:** Armazena todos os dados de configuração e de monitorização recolhidos pelo Zabbix.

2.  **Servidor de Aplicação (`zabbix-server`)**
    * **IP:** `192.168.3.111`
    * **Software:** Zabbix Server 7.0 LTS
    * **Função:** O núcleo do Zabbix. Responsável por recolher os dados, processar triggers e enviar notificações.

3.  **Servidor Web (`zabbix-frontend`)**
    * **IP:** `192.168.3.112`
    * **Software:** Apache2, PHP, Zabbix Frontend
    * **Função:** Fornece a interface web para visualização, configuração e gestão do Zabbix.

## Pré-requisitos

Antes de começar, certifique-se de que tem o seguinte software instalado na sua máquina:

* [**Vagrant**](https://www.vagrantup.com/downloads)
* [**VirtualBox**](https://www.virtualbox.org/wiki/Downloads)

## Como Usar

1.  **Clonar o Repositório**
    Clone este repositório para a sua máquina local.
    ```bash
    git clone <url-do-seu-repositorio>
    cd <nome-do-repositorio>
    ```

2.  **Iniciar o Ambiente**
    Execute o seguinte comando para que o Vagrant crie, configure e inicie as três máquinas virtuais.
    ```bash
    vagrant up
    ```
    > **Nota:** Na primeira execução, o Vagrant irá pedir-lhe para escolher uma interface de rede para o modo "bridge". Selecione a interface que o seu computador utiliza para se conectar à sua rede local (geralmente a sua placa de rede principal, como `enp6s0` ou `wlan0`).

3.  **Configuração do Frontend (Passo Manual)**
    Após o `vagrant up` ser concluído, o ambiente estará pronto, mas precisa de configurar a conexão do frontend através do assistente web.

    * Abra o seu navegador e aceda a: **http://192.168.3.112/zabbix**

    * Siga os passos do assistente de instalação. Na etapa **"Configure DB connection"**, utilize os seguintes dados:
        * **Database type:** `PostgreSQL`
        * **Database host:** `192.168.3.110`
        * **Database port:** `5432`
        * **Database name:** `zabbix`
        * **User:** `zabbix`
        * **Password:** `your_secure_password` (ou a que definiu no `Vagrantfile`)

    * Na etapa seguinte, **"Zabbix server details"**, confirme os dados:
        * **Host:** `192.168.3.111`
        * **Port:** `10051`

    * Conclua os restantes passos do assistente.

### Troubleshooting: Erro de Conexão com o Servidor Zabbix

Se, após a configuração, o dashboard do Zabbix mostrar um erro a indicar que não consegue conectar-se ao servidor Zabbix, pode ser necessário definir o endereço do servidor manualmente no ficheiro de configuração. Siga estes passos:

1.  **Aceda à máquina do frontend:**
    ```bash
    vagrant ssh zabbix-frontend
    ```
2.  **Abra o ficheiro de configuração para edição:**
    ```bash
    sudo nano /etc/zabbix/web/zabbix.conf.php
    ```
3.  **Localize e altere as seguintes linhas** (removendo os `//` do início para as descomentar):
    ```php
    // Altere de:
    // $ZBX_SERVER                      = '';
    // $ZBX_SERVER_PORT                 = '';

    // Para:
    $ZBX_SERVER                      = '192.168.3.111';
    $ZBX_SERVER_PORT                 = '10051';
    ```
4.  **Salve o ficheiro e reinicie o Apache:**
    ```bash
    sudo systemctl restart apache2
    ```

## Acesso

* **Frontend Zabbix:** [http://192.168.3.112/zabbix](http://192.168.3.112/zabbix)
    * **Utilizador Padrão:** `Admin`
    * **Senha Padrão:** `zabbix`

* **Acesso SSH às Máquinas:**
    ```bash
    vagrant ssh zabbix-db
    vagrant ssh zabbix-server
    vagrant ssh zabbix-frontend
    ```

## Customização

Pode alterar a senha do banco de dados antes de iniciar o ambiente, editando a seguinte variável no topo do `Vagrantfile`:

```ruby
$db_password = "sua_nova_senha_segura"