# employee-api
## Create AWS EC2 Instance
> AMI: USe Linux based image Ubuntu 22/24
> Type: Use atleast t2.medium(t2.large recommended)
> Storage: Atleast 15 gb
> Security Group Ports: 22 for SSH, 8080 for http (swagger UI) & 9042 for scyllaDB
### SSH into the instance
> sudo apt update : Update your server
## Installing dependencies for employess-api
### Go-lang
> sudo apt install -y golang-go && go version
    *Installs golang and checks version to confirm download*
### ScyllaDB
> sudo mkdir -p /etc/apt/keyrings
    *Creates a new directory (/etc/apt/keyrings) if it doesn't exist, using -p to ensure no errors if the directory already exists.*
> sudo gpg --homedir /tmp --no-default-keyring --keyring /etc/apt/keyrings/scylladb.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys a43e06657bac99e3
    *This imports the ScyllaDB GPG public key into the system’s keyring. The key is used to verify the authenticity of the packages from the ScyllaDB repository. The key a43e06657bac99e3 is fetched from the key server.*
> sudo wget -O /etc/apt/sources.list.d/scylla.list http://downloads.scylladb.com/deb/debian/scylla-6.2.list
    *Downloads the ScyllaDB repository list (scylla.list) and places it in the correct directory (/etc/apt/sources.list.d/). This list contains the URLs of repositories that provide ScyllaDB packages*
> sudo apt update
    *Updates the package list again, this time including the newly added ScyllaDB repository, so that the system is aware of the available packages from ScyllaDB.*
> sudo apt install -y scylla && scylla --version
    *Installs scylla and checks version to confirm download*
### jq
> sudo apt install jq
### make 
> sudo apt install make
### golang-migrate
> curl -L https://github.com/golang-migrate/migrate/releases/download/v4.15.2/migrate.linux-amd64.tar.gz | tar xvz
    *Downloads golang-migrate into current working directory and extracts it*
> sudo mv migrate /usr/local/bin/migrate
    *Moves migrate binary to /usr/local/bin/ so that it can be used system-wide*
## Configure ScyllaDB for CQL authorization
> cd /etc/scylla
> sudo vi scylla.yaml
    *Changes to make*
    > rpc_address:[instance_private_ip]
    > authenticator: PasswordAuthenticator
    > authorizer: CassandraAuthorizer
> sudo /opt/scylladb/scripts/scylla_io_setup
    *Scylla dependencies installation and service file creation*
> sudo systemctl enable scylla-server
> sudo systemctl start scylla-server
## Create KEYSPACE in scylla 
> cqlsh <instance-private-Ip> 9042 -u cassandra -p cassandra
    *Connects to ScyllaDB on the private IP with the username cassandra and password cassandra*
### Inside cqlsh 
> CREATE USER scylladb WITH PASSWORD 'password' SUPERUSER;
    *Creates a user scylladb with superuser privileges and the password password.*
> CREATE KEYSPACE employee_db WITH REPLICATION = { 'class': 'SimpleStrategy', 'replication_factor': 1 };
    *Creates a new keyspace (database) named employee_db with a replication factor of 1.*
>  DESCRIBE KEYSPACES;
    *Lists all available keyspaces in the database.*
### Working with Employee-api git repositary 
> git clone https://github.com/OT-MICROSERVICES/employee-api.git
    *Clones the employee-Api repository using HTTPS*
> cd employee-api && ls
    *Changes the directory to employee-Api and lists its contents*
### Migration configuration
> vi migration.json
    *Replace the ip address with your actual private ip of instance*
> make run-migrations
    *runs the Makefile present in the current woring directiory*
### Config file setting
> vi config.yaml
    *Opens the config.yaml file in text editor. Edit this file to configure settings for the application and write replace the ip with your actual private Ip of the instance in host*
### Testing go modules
> go test $(go list ./... | grep -v docs | grep -v model | grep -v main.go) -coverprofile cover.out
    *Tests main.go*
> go tool cover -html=cover.out
    *Generates html coverage report*
### Configure main.go
> vi main.go'
    *Replace localhost to <public-ip:8080>*
### Run the API
> go run main.go
## Check working on 
http://public-ip:8080/swagger/index.html
