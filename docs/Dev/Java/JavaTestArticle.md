# 使用 systemd 管理 Java 程序

**a simple demo:**

```bash
vim /etc/systemd/system/halo.service
```

```bash
[Unit]
Description=Halo Service
Documentation=https://halo.run
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=USER
ExecStart=/usr/bin/java -server -Xms256m -Xmx256m -jar YOUR_JAR_PATH
ExecStop=/bin/kill -s QUIT $MAINPID
Restart=always
StandOutput=syslog

StandError=inherit

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl start halo
systemctl status halo
systemctl enable halo
journalctl -n 20 -u halo

sudo systemctl restart myapp
```

**another:**

```bash
[Unit]
Description=Manage Java service

[Service]
WorkingDirectory=/opt/prod
ExecStart=/bin/java -Xms128m -Xmx256m -jar myapp.jar
User=jvmapps
Type=simple
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

**one more:**

```bash
[Unit]
Description=My Java driven simple service
After=syslog.target network.target

[Service]
SuccessExitStatus=143

User=appuser
Group=appgroup

Type=simple

Environment="JAVA_HOME=/path/to/jvmdir"
WorkingDirectory=/path/to/app/workdir
ExecStart=${JAVA_HOME}/bin/java -jar javaapp.jar
ExecStop=/bin/kill -15 $MAINPID

[Install]
WantedBy=multi-user.target
```

- ExecStop specifies the service termination command, and systemd is smart enough to figure out the PID of the started process. It automatically creates a MAINPID environment variable
- systemd to send a 15 (SIGTERM) system signal to terminate the process gracefully.
- The JVM designers made Java return a non-zero exit code in case it is terminated by a system signal. As a non-zero base, they took 128, and the resulting exit code is a sum of 128 and the signal numeric value.
- ***By setting SuccessExitStatus to 143, we tell systemd to handle that value (128+15) as a normal exit.***

**forking mode:**

use a shell script to start the service, the JVM will be started by the shell (bash) process. This operation is known as fork, and it’s why we set the service Type to forking.

```bash
#!/bin/bash

JAVA_HOME=/path/to/jvmdir
WORKDIR=/path/to/app/workdir
JAVA_OPTIONS=" -Xms256m -Xmx512m -server "
APP_OPTIONS=" -c /path/to/app.config -d /path/to/datadir "

cd $WORKDIR
"${JAVA_HOME}/bin/java" $JAVA_OPTIONS -jar javaapp.jar $APP_OPTIONS
```

```bash
[Unit]
Description=My Java forking service
After=syslog.target network.target
[Service]
SuccessExitStatus=143
User=appuser
Group=appgroup

Type=forking

ExecStart=/path/to/wrapper
ExecStop=/bin/kill -15 $MAINPID

[Install]
WantedBy=multi-user.target
```

TODO：

- after
- wanted by

## REF

- [在 Centos 7.x 上安装 | Halo Documents](https://docs.halo.run/zh/install/centos)
- [Run a Java Application as a Service on Linux | Baeldung on Linux](https://www.baeldung.com/linux/run-java-application-as-service)
