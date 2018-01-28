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

Find out the name of the container:

```
[root@server tmp]# docker ps
CONTAINER ID        IMAGE                                  COMMAND                  CREATED             STATUS                  PORTS                                                           NAMES
bcec9674d430        jpdurot/opentsdb:latest                "/run.sh"                5 hours ago         Up 5 hours                                                                              sharp_golick
                                                            sharp_golick
```

Now load up a bash shell within the container so we can make some metrics. eos.counters.rates(bitsrate|pktsrate) is used in the sample.json file. 

```
docker exec -t -i sharp_golick /bin/bash
/usr/share/opentsdb/bin/tsdb mkmetric eos.counters.rates.bitsrate
/usr/share/opentsdb/bin/tsdb mkmetric eos.counters.rates.pktsrate
```

Finally, resolve the IP address of the OpenTSDB container:

```
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


# sample.json file

```
{
        "comment": "This is a sample configuration",
        "subscriptions": [
                "/Smash/counters/ethIntf/PhyEthtool-1/current/"
        ],
        "metricPrefix": "eos",
        "metrics": {
                "test": {
                        "path": "/Smash/(counters)/ethIntf/PhyEthtool-1/current/counter/(?P<intf>.+)/(rates)/(?P<direction>(?:in|out))(BitsRate|PktsRate)"
                }
        }
}
```

This gives a simple overview of how the translation of the OpenConfig to OpenTSDB metrics works.

 - metricPrefix is the starting name of the metrics
 - Anything in parentheses "()" are added to the metric patch
 - Anything in chevrons "<>" are tags

So in the above example:
 - the metrics will be eos.counters.rates.bitsrate & eos.counters.rates.bitsrate.
 - the tages will be the interface name (<intf>) and the direction (<direction>(?:in|out))
