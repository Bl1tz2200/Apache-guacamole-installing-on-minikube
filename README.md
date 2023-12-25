# Apache-guacamole-installing-on-minikube
I created an apache guacamole service using minikube and kubertenes utilites.<br />
So let's get start

# First things First
We need to install minikube and his controlling utilites: kubectl, kubeadm, kubelet and docker.<br />
Also we need to install mysql-server by:

$ sudo apt-get install -y mysql-server

(You can use PostreSql too, but i will use MySql, that's why you will need to change many settings in yaml files)

To install minikube you should use:

$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64<br />
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube
	
To to install kubectl, kubeadm and kubelet you can go to k8s site:

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

If you get ERRORS after trying to install by instruction of k8s try:

$ snap install kubelet<br />
$ snap install kubeadm<br />
$ snap install kubectl

To install docker you can just:

$ snap install docker

After successful installation we can start

# Let's set up out DB
As i said I will use MySqlDB

First we need to use this command: (I got it from https://guacamole.apache.org/doc/gug/guacamole-docker.html#guacamole-docker-mysql)

$ docker run --rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --mysql > initdb.sql

It will get configuration file that will make mysql tables for guacamole

After initialazing root using:

$ sudo mysql_secure_installation

We should create DB named guacamole_db:

$ mysql -u root -p<br />
mysql> CREATE DATABASE IF NOT EXISTS guacamole_db;

We should create user guacamole_user with password guacpassword and privileges to change guacamole_db:

mysql> CREATE USER 'guacamole_user' IDENTIFIED BY 'guacpassword';<br />
mysql> GRANT ALL ON guacamole_db.* TO 'guacamole_user';

(you can change user and password, but don`t forget to change it in your guacamole_secret.yaml configuration (remember: values in secret uses Base64 coding, you should code your user and login in to their Base64 variation)

After that we use already installed initdb:

mysql> USE guacamole_db;<br />
mysql> source /your/path/to/initdb.sql;<br />
mysql> exit

Don't forget to change ip address for your mysql server from localhost to another in /etc/mysql/mysql.conf.d/mysqld.cnf:

...<br />
bind-address=0.0.0.0 #You should use another ip address because listening all ip addresses isn't safe<br />
mysqlx-bind-address=0.0.0.0<br />
...

And to change it in guacamole_entrypoint_for_db_service.yaml

And we'll get working mysql DB for guacamole.

# Close to end
Initialization of DB was the hardest part<br />
Now move all files that I attached to GitHub to one directory<br />
We should start minikube:

$ minikube start --driver=docker

Wait couple of minutes and after installation you can run:

$ for i in *.yaml; do kubectl apply -f $i; done

Another waiting, you can check the progress using:

$ kubectl get pod<br />

If all pods has 1/1 without restarts it's great<br />
Now run:

$ minikube service apache-guacamole-main-service

And you will get the link to your site. If you add /guacamole (https://YOUR_IP_ADDRESS_WITH_PORT_SHOWED_IN_TERMINAL/guacamole) you will get in site.<br />
Basicaly Apache Guacamole has guacadmin on username and login.<br />
Now you can start working







