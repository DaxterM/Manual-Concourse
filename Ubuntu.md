# manualconcourse

# Install Postgress
```
$ sudo apt-get install postgresql postgresql-contrib -y
$ sudo -u postgres createuser concourse
$ sudo -u postgres createdb --owner=concourse atc
```
# Install concourse
```
$ wget https://github.com/concourse/concourse/releases/download/v3.3.4/concourse_linux_amd64
$ chmod +x concourse_linux_amd64 
$ sudo mv concourse* /usr/local/bin/concourse
$ sudo mkdir /etc/concourse
$ sudo adduser --system --group concourse
$ sudo chown -R concourse:concourse /etc/concourse
```
# Create keys
```

$ sudo ssh-keygen -t rsa -q -N '' -f /etc/concourse/tsa_host_key
$ sudo ssh-keygen -t rsa -q -N '' -f /etc/concourse/worker_key
$ sudo ssh-keygen -t rsa -q -N '' -f /etc/concourse/session_signing_key
$ sudo cp /etc/concourse/worker_key.pub /etc/concourse/authorized_worker_keys
$ sudo chown -R concourse:concourse /etc/concourse
```

# Configure Concourse Web
```
$ sudo vi /etc/concourse/web_environment
CONCOURSE_SESSION_SIGNING_KEY=/etc/concourse/session_signing_key
CONCOURSE_TSA_HOST_KEY=/etc/concourse/tsa_host_key
CONCOURSE_TSA_AUTHORIZED_KEYS=/etc/concourse/authorized_worker_keys
CONCOURSE_POSTGRES_SOCKET=/var/run/postgresql

CONCOURSE_BASIC_AUTH_USERNAME=admin
CONCOURSE_BASIC_AUTH_PASSWORD=P@ssw0rd
CONCOURSE_EXTERNAL_URL=http://IP_HERE:8080

$ sudo vi /etc/systemd/system/concourse-web.service

[Unit]
Description=Concourse CI web process (ATC and TSA)
After=postgresql.service

[Service]
User=concourse
Restart=on-failure
EnvironmentFile=/etc/concourse/web_environment
ExecStart=/usr/local/bin/concourse web

[Install]
WantedBy=multi-user.target
$ sudo chown -R concourse:concourse /etc/concourse
```


# Configure Concourse Worker
```
$ sudo vi /etc/concourse/worker_environment

CONCOURSE_WORK_DIR=/var/lib/concourse
CONCOURSE_TSA_WORKER_PRIVATE_KEY=/etc/concourse/worker_key
CONCOURSE_TSA_PUBLIC_KEY=/etc/concourse/tsa_host_key.pub
CONCOURSE_TSA_HOST=127.0.0.1

$ sudo vi /etc/systemd/system/concourse-worker.service

[Unit]
Description=Concourse CI worker process
After=concourse-web.service

[Service]
User=root
Restart=on-failure
EnvironmentFile=/etc/concourse/worker_environment
ExecStart=/usr/local/bin/concourse worker

[Install]
WantedBy=multi-user.target

$ sudo chown -R concourse:concourse /etc/concourse
```


# Test
```
$ sudo systemctl start concourse-web concourse-worker
$ sudo systemctl status concourse-web concourse-worker

$ curl http://IP:8080
Check in browser
```

# Enable systemctl scripts to start on reboot
```
sudo systemctl enable concourse-web concourse-worker
```

