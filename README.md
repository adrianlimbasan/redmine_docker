# Redmine on Docker

## Requirements

### Linux
1. Git
2. Docker
3. Docker Compose

### Windows

1. Git
2. VirtualBox
3. Vagrant 
4. Vagrant NFS plugin: `vagrant plugin install vagrant-winnfsd`

## Install

1. Add the following to your `~/.bashrc` file:

    ```bash
    export HOST_USER_UID=`id -u`
    export HOST_USER_GID=`id -g`

    alias docker-exec='docker exec -it -u `id -u`:`id -g`'
    alias docker-tail='docker logs -f --tail=100'
    ```

2. Source the new `~/.bashrc` file:

    ```bash
    source ~/.bash_profile
    ```

1. Clone this repository:

    ``` bash
    git clone https://github.com/marius-balteanu/redmine_docker.git
    ```

2. Go inside the `redmine_docker/project` directory and clone the redmine repository:

    ```bash
    cd redmine_docker/project

    # Using HTTPS
    git clone https://github.com/redmine/redmine.git

    # Using SSH
    git clone git@github.com:redmine/redmine.git
    ```

3. Add a file named `env.list` in the `secrets` directory.
See `/secrets/.env.list.sample` for what it should contain.

4. To build the containers run:

    ```bash
    docker-compose -f docker/docker-compose.build.yml build
    docker-compose -f docker/docker-compose.build-dev.yml build
    ```

4. To start the containers run:

    ```bash
    docker-compose -f docker/docker-compose.yml up -d
    ```

5. To set up a fresh database run:

    ```bash
    docker-exec redmine_web_1 /entry \
      bundle exec rake db:create db:migrate redmine:load_default_data REDMINE_LANG=en
    ```
    
6. To reload the entire application run:

    ```bash
    docker-compose restart
    ```
    
## View logs

```bash
docker-tail redmine_web_1
```

## Execute commands inside a contaier

Make sure it is running and then run:

```bash
docker-exec redmine_web_1 /entry bundle install
```
