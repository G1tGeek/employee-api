# employee-api
## Always view this file in RAW format present near upper right corner of github page

## Create AWS EC2 Instance
- AMI: Use Linux based image Ubuntu 22/24  
- Type: Use at least t2.medium (t2.large recommended)  
- Storage: At least 15 GB  
- Security Group Ports: 22 for SSH, 8080 for HTTP (Swagger UI) & 9042 for ScyllaDB  

### SSH into the instance
- sudo apt update
  > *Update your server*

## Installing dependencies for employee-api

### Go-lang
- cd /tmp
- wget https://go.dev/dl/go1.21.5.linux-amd64.tar.gz
- sudo tar -C /usr/local -xzf go1.21.5.linux-amd64.tar.gz
- export PATH=$PATH:/usr/local/go/bin
- source ~/.bashrc
- go version


### ScyllaDB
- sudo mkdir -p /etc/apt/keyrings
  > *Creates a new directory (/etc/apt/keyrings) if it doesn't exist, using -p to ensure no errors if the directory already exists.*

- sudo gpg --homedir /tmp --no-default-keyring --keyring /etc/apt/keyrings/scylladb.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys a43e06657bac99e3
  > *Imports the ScyllaDB GPG public key into the system’s keyring.*

- sudo wget -O /etc/apt/sources.list.d/scylla.list http://downloads.scylladb.com/deb/debian/scylla-6.2.list
  > *Adds the ScyllaDB repository.*

- sudo apt update
  > *Updates the package list with ScyllaDB sources.*

- sudo apt install -y scylla && scylla --version
  > *Installs ScyllaDB and checks the version.*

### jq
- sudo apt install jq

### make
- sudo apt install make

### golang-migrate
- curl -L https://github.com/golang-migrate/migrate/releases/download/v4.15.2/migrate.linux-amd64.tar.gz | tar xvz
  > *Downloads golang-migrate and extracts it.*

- sudo mv migrate /usr/local/bin/migrate
  > *Moves the migrate binary to a system-wide location.*

## Configure ScyllaDB for CQL authorization
- cd /etc/scylla  
- sudo vi scylla.yaml  

*Changes to make:*  
- listen_address: private ip
- seeds: private ip
- rpc_address: instance_private_ip
> Add  
- authenticator: PasswordAuthenticator  
- authorizer: CassandraAuthorizer

- sudo /opt/scylladb/scripts/scylla_io_setup
  > *Scylla dependencies installation and service file creation*

- sudo systemctl enable scylla-server  
- sudo systemctl start scylla-server

## Create KEYSPACE in Scylla
- cqlsh <instance-private-IP> 9042 -u cassandra -p cassandra
  > *Connects to ScyllaDB*

### Inside cqlsh
- CREATE USER scylladb WITH PASSWORD 'password' SUPERUSER;  
- CREATE KEYSPACE employee_db WITH REPLICATION = { 'class': 'SimpleStrategy', 'replication_factor': 1 };  
- DESCRIBE KEYSPACES;

## Working with employee-api Git repository
- git clone https://github.com/OT-MICROSERVICES/employee-api.git  
- cd employee-api && ls

### Migration configuration
- vi migration.json
  > *Replace the IP address with your actual private IP*

- make run-migrations

### Config file setting
- vi config.yaml
  > *Replace the IP address with your actual private IP*

### Testing Go modules
- go test $(go list ./... | grep -v docs | grep -v model | grep -v main.go) -coverprofile cover.out  
- go tool cover -html=cover.out

### Configure main.go
- vi main.go
  > *Replace `localhost` with `<public-ip:8080>`*

### Run the API
- go run main.go

## Check working on  
[http://public-ip:8080/swagger/index.html](http://public-ip:8080/swagger/index.html)
