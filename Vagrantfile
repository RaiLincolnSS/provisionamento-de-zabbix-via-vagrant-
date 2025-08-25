# -*- mode: ruby -*-
# vi: set ft=ruby :

# Senha para o banco de dados. Altere se desejar.
$db_password = "Banco123"

Vagrant.configure("2") do |config|
  
  # -------------------------------------------------------------------
  # 1. Servidor de Banco de Dados (PostgreSQL)
  # -------------------------------------------------------------------
  config.vm.define "zabbix-db" do |db|
    # Alterado para Ubuntu 22.04 LTS (Jammy Jellyfish) para garantir a disponibilidade da box
    db.vm.box = "ubuntu/jammy64"
    db.vm.hostname = "zabbix-db"
    # Alterado para public_network para conectar à sua rede local
    db.vm.network "public_network", ip: "192.168.3.110"

    db.vm.provider "virtualbox" do |v|
      v.memory = 1024
      v.cpus = 1
      v.name = "zabbix-db-server"
    end

    db.vm.provision "shell", inline: <<-SHELL
      echo ">>> [DB] Atualizando pacotes..."
      apt-get update -y > /dev/null

      echo ">>> [DB] Instalando PostgreSQL..."
      apt-get install -y postgresql

      echo ">>> [DB] Configurando PostgreSQL para aceitar conexões externas..."
      # Caminho do arquivo de configuração atualizado para a versão 14 do PostgreSQL (padrão no Ubuntu 22.04)
      sed -i "s/#listen_addresses = 'localhost'/listen_addresses = '*'/" /etc/postgresql/14/main/postgresql.conf
      
      # Adiciona uma regra no pg_hba.conf para permitir a conexão do Zabbix Server e do Frontend
      echo "host    zabbix          zabbix          192.168.3.111/32        md5" >> /etc/postgresql/14/main/pg_hba.conf
      echo "host    zabbix          zabbix          192.168.3.112/32        md5" >> /etc/postgresql/14/main/pg_hba.conf

      echo ">>> [DB] Reiniciando PostgreSQL..."
      systemctl restart postgresql

      echo ">>> [DB] Criando usuário e banco de dados para o Zabbix..."
      # Cria o usuário 'zabbix' com a senha definida no início do script
      sudo -u postgres psql -c "CREATE USER zabbix WITH PASSWORD '#{$db_password}';"
      # Cria o banco de dados 'zabbix' e define o 'zabbix' como proprietário
      sudo -u postgres psql -c "CREATE DATABASE zabbix WITH OWNER zabbix;"
      
      echo ">>> [DB] Provisionamento concluído!"
    SHELL
  end

  # -------------------------------------------------------------------
  # 2. Servidor de Aplicação (Zabbix Server)
  # -------------------------------------------------------------------
  config.vm.define "zabbix-server" do |server|
    # Alterado para Ubuntu 22.04 LTS (Jammy Jellyfish)
    server.vm.box = "ubuntu/jammy64"
    server.vm.hostname = "zabbix-server"
    # Alterado para public_network para conectar à sua rede local
    server.vm.network "public_network", ip: "192.168.3.111"
    
    # O servidor do Zabbix depende que o banco de dados esteja no ar
    server.vm.provision "shell", run: "once", inline: "echo 'Aguardando o DB iniciar...'; sleep 15"

    server.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
      v.name = "zabbix-app-server"
    end

    server.vm.provision "shell", inline: <<-SHELL
      echo ">>> [SERVER] Atualizando pacotes..."
      apt-get update -y > /dev/null

      echo ">>> [SERVER] Baixando e instalando o repositório Zabbix..."
      apt-get install -y wget
      # URL do repositório Zabbix atualizada para a versão do Ubuntu 22.04
      wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu22.04_all.deb
      dpkg -i zabbix-release_7.0-1+ubuntu22.04_all.deb
      apt-get update -y > /dev/null

      echo ">>> [SERVER] Instalando Zabbix Server e cliente PostgreSQL..."
      apt-get install -y zabbix-server-pgsql zabbix-sql-scripts postgresql-client

      echo ">>> [SERVER] Importando o schema inicial do Zabbix no banco de dados..."
      # A variável PGPASSWORD é usada para fornecer a senha de forma não interativa
      zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | PGPASSWORD='#{$db_password}' psql -h 192.168.3.110 -U zabbix zabbix

      echo ">>> [SERVER] Configurando a conexão com o banco de dados..."
      # Edita o arquivo de configuração do Zabbix Server para apontar para o IP do DB
      sed -i 's/^# DBHost=localhost/DBHost=192.168.3.110/' /etc/zabbix/zabbix_server.conf
      sed -i "s/^# DBPassword=/DBPassword=#{$db_password}/" /etc/zabbix/zabbix_server.conf

      echo ">>> [SERVER] Iniciando e habilitando o serviço Zabbix Server..."
      systemctl restart zabbix-server
      systemctl enable zabbix-server
      
      echo ">>> [SERVER] Provisionamento concluído!"
    SHELL
  end

  # -------------------------------------------------------------------
  # 3. Servidor Web (Zabbix Frontend)
  # -------------------------------------------------------------------
  config.vm.define "zabbix-frontend" do |frontend|
    # Alterado para Ubuntu 22.04 LTS (Jammy Jellyfish)
    frontend.vm.box = "ubuntu/jammy64"
    frontend.vm.hostname = "zabbix-frontend"
    # Alterado para public_network para conectar à sua rede local
    frontend.vm.network "public_network", ip: "192.168.3.112"
    
    # Expõe a porta 80 da VM para a porta 8080 da sua máquina local
    frontend.vm.network "forwarded_port", guest: 80, host: 8080, auto_correct: true

    frontend.vm.provider "virtualbox" do |v|
      v.memory = 1024
      v.cpus = 1
      v.name = "zabbix-frontend-server"
    end

    frontend.vm.provision "shell", inline: <<-SHELL
      echo ">>> [WEB] Atualizando pacotes..."
      apt-get update -y > /dev/null

      echo ">>> [WEB] Baixando e instalando o repositório Zabbix..."
      apt-get install -y wget
      # URL do repositório Zabbix atualizada para a versão do Ubuntu 22.04
      wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu22.04_all.deb
      dpkg -i zabbix-release_7.0-1+ubuntu22.04_all.deb
      apt-get update -y > /dev/null

      echo ">>> [WEB] Instalando o Zabbix Frontend, Apache e o Zabbix Agent..."
      # Versão do PHP atualizada para 8.1 (padrão no Ubuntu 22.04)
      apt-get install -y zabbix-frontend-php php8.1-pgsql zabbix-apache-conf zabbix-agent

      echo ">>> [WEB] Configurando o Zabbix Agent para reportar ao Zabbix Server..."
      sed -i 's/^Server=127.0.0.1/Server=192.168.3.111/' /etc/zabbix/zabbix_agentd.conf
      sed -i 's/^ServerActive=127.0.0.1/ServerActive=192.168.3.111/' /etc/zabbix/zabbix_agentd.conf

      echo ">>> [WEB] Iniciando e habilitando os serviços..."
      systemctl restart apache2 zabbix-agent
      systemctl enable apache2 zabbix-agent
      
      echo ">>> [WEB] Provisionamento concluído!"
    SHELL
  end
end
