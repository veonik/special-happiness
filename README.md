special-happiness
=================

Dockerized GitLab, Redis, and Redis Sentinel.


Overview
--------

This docker-compose will setup GitLab 9 CE, Redis, Redis Sentinel, and MailHog. It also allows for bringing up slave Redis/Sentinel containers.

GitLab is configured to use Redis Sentinel to failover to one of the slave Redis containers in the event the current master fails.


Usage
-----

> These instructions presume you already have Docker and docker-compose installed on your machine.

[Download the source code for this repository](https://github.com/veonik/special-happiness/archive/master.zip) or clone the project. 

From the project's root directory, start everything with `docker-compose up`.

```bash
docker-compose up
```

After a time, you should see happy little GitLab messages and can visit the application in your browser.

GitLab will be available in your browser at `http://localhost:8082`. You should be presented with setting the `root` user's password.

Login with the username `root` and whatever password you selected.

### Starting additional Redis containers

Use `docker-compose scale` to start as many slave Redis containers as you like.

For example, to start four (4) Redis slaves, run:

```bash
docker-compose scale redis-slave=4
```

### Testing Redis Sentinel

The simplest way to see what Sentinel is up to is to start a shell on one of the Redis containers and then connect to Sentinel using `redis-cli` and the `INFO` command.

```bash
$ docker exec -it specialhappiness_redis-master_1 /bin/bash
root@979a3c28b39e:/data# redis-cli -p 26379
127.0.0.1:26379> INFO
#.... some output
master0:name=mymaster,status=ok,address=172.18.0.3:6379,slaves=4,sentinels=5
```

### Testing Redis failover

After you've verified Sentinel is working, you can test failover by killing the master, or by using Redis's `DEBUG sleep` command.

```bash
docker exec -t specialhappiness_redis-master_1 redis-cli -p 6379 DEBUG sleep 30
```

After a few seconds, your console should light up with activity from the Redis containers. You can verify that the master changed by [running the `INFO` command on one of the Redis Sentinel instances](#testing-redis-sentinel).

### Testing GitLab failover

Follow the steps outlined above, then look for some output from the gitlab container regarding the failover.


```
# ... lots more stacks
gitlab_1        | 2017-10-25_03:55:25.96920 2017-10-25T03:55:25.967Z 746 TID-ow9t2ls3c ERROR: /opt/gitlab/embedded/lib/ruby/gems/2.3.0/gems/sidekiq-5.0.4/lib/sidekiq.rb:92:in `redis'
gitlab_1        | 2017-10-25_03:55:25.96921 2017-10-25T03:55:25.967Z 746 TID-ow9t2ls3c ERROR: /opt/gitlab/embedded/lib/ruby/gems/2.3.0/gems/sidekiq-5.0.4/lib/sidekiq/fetch.rb:36:in `retrieve_work'
gitlab_1        | 2017-10-25_03:55:25.96931 2017-10-25T03:55:25.967Z 746 TID-ow9t2ls3c ERROR: /opt/gitlab/embedded/lib/ruby/gems/2.3.0/gems/sidekiq-5.0.4/lib/sidekiq/processor.rb:91:in `get_one'
gitlab_1        | 2017-10-25_03:55:25.96938 2017-10-25T03:55:25.967Z 746 TID-ow9t2ls3c ERROR: /opt/gitlab/embedded/lib/ruby/gems/2.3.0/gems/sidekiq-5.0.4/lib/sidekiq/processor.rb:101:in `fetch'
gitlab_1        | 2017-10-25_03:55:25.96942 2017-10-25T03:55:25.968Z 746 TID-ow9t2ls3c ERROR: /opt/gitlab/embedded/lib/ruby/gems/2.3.0/gems/sidekiq-5.0.4/lib/sidekiq/processor.rb:84:in `process_one'
gitlab_1        | 2017-10-25_03:55:25.96945 2017-10-25T03:55:25.968Z 746 TID-ow9t2ls3c ERROR: /opt/gitlab/embedded/lib/ruby/gems/2.3.0/gems/sidekiq-5.0.4/lib/sidekiq/processor.rb:73:in `run'
gitlab_1        | 2017-10-25_03:55:25.96952 2017-10-25T03:55:25.968Z 746 TID-ow9t2ls3c ERROR: /opt/gitlab/embedded/lib/ruby/gems/2.3.0/gems/sidekiq-5.0.4/lib/sidekiq/util.rb:16:in `watchdog'
gitlab_1        | 2017-10-25_03:55:25.96964 2017-10-25T03:55:25.968Z 746 TID-ow9t2ls3c ERROR: /opt/gitlab/embedded/lib/ruby/gems/2.3.0/gems/sidekiq-5.0.4/lib/sidekiq/util.rb:25:in `block in safe_thread'
redis-slave_2   | 10:X 25 Oct 03:55:26.385 # +sdown slave 172.18.0.3:6379 172.18.0.3 6379 @ mymaster 172.18.0.4 6379
redis-slave_3   | 9:X 25 Oct 03:55:26.454 # +sdown slave 172.18.0.3:6379 172.18.0.3 6379 @ mymaster 172.18.0.4 6379
redis-slave_4   | 10:X 25 Oct 03:55:26.412 # +sdown slave 172.18.0.3:6379 172.18.0.3 6379 @ mymaster 172.18.0.4 6379
redis-slave_1   | 10:X 25 Oct 03:55:26.443 # +sdown slave 172.18.0.3:6379 172.18.0.3 6379 @ mymaster 172.18.0.4 6379
gitlab_1        | 2017-10-25_03:55:28.04403 2017-10-25T03:55:28.043Z 746 TID-ow9t2lsk0 INFO: Redis is online, 3.495915106 sec downtime
gitlab_1        | 2017-10-25_03:55:28.24597 2017-10-25T03:55:28.245Z 746 TID-ow9t2lb7k INFO: Redis is online, 3.49493719 sec downtime
# ...
```

### Cleaning up

Run `docker-compose down` to turn off and delete all of the containers that were started.

```bash
docker-compose down
```

# Notes

GitHub suggested the name, and I liked it.

Special thanks to [Aeolun](https://github.com/Aeolun) for the original contribution this is based on.
