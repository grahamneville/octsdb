# octsdb


This is a (64-bit) compiled version of Arista's octsdb: https://github.com/aristanetworks/goarista/tree/master/cmd/octsdb

This subscribes to TerminAttr - gRPC OpenConfig - and writes the metrics to OpenTSDB

# Example usage:

This explains how to get a POC up and running using docker

Download and run OpenTSDB docker image:

```
docker pull jpdurot/opentsdb
docker run -d -p 4242:4242 jpdurot/opentsdb:latest
```

Now load up a bash shell within the container so we can make some metrics.

```
docker exec -t -i sharp_golick /bin/bash
/usr/share/opentsdb/bin/tsdb mkmetric eos.status
```

Finally, find the name of the container and resolve the IP address of the OpenTSDB container:

```
[root@server tmp]# docker ps
CONTAINER ID        IMAGE                                  COMMAND                  CREATED             STATUS                  PORTS                                                           NAMES
bcec9674d430        jpdurot/opentsdb:latest                "/run.sh"                5 hours ago         Up 5 hours                                                                              sharp_golick
                                                            sharp_golick

docker inspect --format '{{.NetworkSettings.IPAddress}}' sharp_golick
```

Now within the Arista switch go in to bash and running the following command:

```
enable
bash
sudo su
/usr/bin/TerminAttr -grpcaddr 0.0.0.0:6042 -allowed_ips 0.0.0.0/0 -disableaaa
```

Finally, on the host running with go installed obtain the octsdb binary and sample.json file

```
git clone https://github.com/grahamneville/octsdb.git
```

Now run the octsdb binary filling in the Arista switch ip and OpenTSDB container IP addresses:

```
./octsdb -addrs <arista_switch_ip>:6042 -config sample.json -v 4 -tsdb <opentsdb_container_ip>:4242
```







