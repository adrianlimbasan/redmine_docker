# Host Setup

## Requirements

1. [Git](https://git-scm.com/download/linux)
2. [Docker](https://docs.docker.com/engine/installation/)
3. [Docker Compose](https://docs.docker.com/compose/install/)

## Initial Setup

1. Go inside the `project` directory from te repository root:
    ```bash
    cd project
    ```

2. Build the containers:
    ```bash
    # First you build the production containers:
    docker-compose -f docker/build/build.yml build

    # If you want to do development you will have to execute the follwoing
    # command as well, otherwise skip it:
    docker-compose -f docker/build/build-dev.yml build
    ```

3. Make sure the reverse proxy is started:
    ```bash
    docker start nginx_reverse_proxy
    ```

4. Start the containers:

    For Development execute just this command below:
    ```bash
    docker-compose -p redmine up -d
    ```

    **Please skip to the next step and DO NOT execute the commands below if you
    do not intend to bring up the PRODUCTION environment**

    For Production you will have to prefix all your commands with the
    `DEPLOYMENT_ENV=production` environment variable.
    ```bash
    DEPLOYMENT_ENV=production docker-compose -p redmine up -d
    ```

    If you do not run multiple environments on the same machine you can also
    export the variable and just use the normal commands:
    ```bash
    export DEPLOYMENT_ENV=production

    docker-compose -p redmine up -d
    ```

    If you want the `DEPLOYMENT_ENV` variable to be permanent you need to set it
    in your `~/.bashrc` file:
    ```bash
    cat <<- 'EOT' >> ~/.bashrc
    export DEPLOYMENT_ENV=production
    EOT

    source ~/.bashrc
    ```

    The possible values for the `DEPLOYMENT_ENV` variable are:
    * `DEPLOYMENT_ENV=development`
    * `DEPLOYMENT_ENV=production`

5. Connect the frontend network to the nginx reverse proxy:
    ```bash
    docker network connect redmine_frontend_network nginx_reverse_proxy
    ```

6. Initialize the database:
    ```bash
    docker-compose -p redmine exec web \
      /entry bundle exec rake db:create db:migrate
    ```

7. Restart the web service:
    ```bash
    docker-compose -p redmine restart web
    ```

8. Visit the application url in a browser:
    ```bash
    http://redmine.local/
    ```


## Cheatsheet

### Execute commands inside a contaier

Make sure the container is running and execute:

```bash
# To install gems use the version without `/enrty`
docker-compose -p redmine exec web bundle install

# Any other command that depends on the enviroment must pass trough `/entry`
docker-compose -p redmine exec web /entry rails c
```


### View logs

You can use the `docker-tail` alias:
```bash
docker-tail redmine_web_1
```

You can also use the full command:
``` bash
docker logs -f --tail=100 redmine_web_1
```


### Debugging

Place a `binding.pry` call where you would like the application to breakpoint.
Then attach to the container:

```bash
docker attach redmine_web_1
```

To detach from the container press `Ctrl+p Ctrl+q`.

>
#### NOTE!
If you press `Ctrl+c` while attached you will terminate the process and stop
the container. You will have to restart the container afterwards.
