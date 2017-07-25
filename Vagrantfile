Vagrant.configure("2") do |config|
  config.vm.box = "mkieboom/mapr-spark-drill"

  config.vm.network "forwarded_port", guest: 8079, host: 8079, id: "NiFi"
  config.vm.network "forwarded_port", guest: 8400, host: 8400, id: "Kylo"

  #config.vm.provision "file", source: "resources", destination: "resources"

  config.vm.provision "shell", inline: <<-SHELL
    sudo yum update -y
    sudo yum install mysql-server -y
    sudo service mysqld start

    mysql -u root -e "UPDATE mysql.user SET Password = PASSWORD('hadoop') WHERE User = 'root'; DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1'); DROP DATABASE IF EXISTS test; FLUSH PRIVILEGES;"

    wget https://s3-us-west-2.amazonaws.com/kylo-io/releases/rpm/0.8.2/kylo-0.8.2-1.noarch.rpm

    sudo useradd -r -m -s /bin/bash nifi
    sudo useradd -r -m -s /bin/bash kylo
    sudo useradd -r -m -s /bin/bash activemq

    sudo rpm -ivh kylo-0.8.2-1.noarch.rpm

    sudo groupadd supergroup
    sudo usermod -a -G supergroup nifi
    sudo usermod -a -G supergroup root

    sudo mkdir -p /var/dropzone
    sudo chmod 774 /var/dropzone/
    sudo chown nifi /var/dropzone

    sudo /opt/kylo/setup/sql/mysql/setup-mysql.sh localhost root hadoop
    
    sudo /opt/kylo/setup/elasticsearch/install-elasticsearch.sh

    sudo service elasticsearch start

    sudo /opt/kylo/setup/activemq/install-activemq.sh /opt/activemq activemq users

    sudo /opt/kylo/setup/java/install-java8.sh

    sudo /opt/kylo/setup/nifi/install-nifi.sh /opt/nifi nifi users

    sudo /opt/kylo/setup/java/change-nifi-java-home.sh /opt/java/current /opt/nifi/current/

    sudo /opt/kylo/setup/nifi/install-kylo-components.sh /opt/nifi /opt/kylo nifi users

    rm kylo-0.8.2-1.noarch.rpm

    sudo /opt/kylo/start-kylo-apps.sh
    sudo service nifi start
  SHELL
end
