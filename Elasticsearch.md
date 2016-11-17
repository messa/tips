Elasticsearch tips
==================

Installation on Debian
----------------------

Documentation: [Install Elasticsearch with Debian Package](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/deb.html)

```shell
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" \
    | sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list
sudo apt-get update
sudo apt-get install elasticsearch
```

Troubleshooting: Java 8 required
--------------------------------

When starting:

```shell
$ /usr/share/elasticsearch/bin/elasticsearch --help
Exception in thread "main" java.lang.UnsupportedClassVersionError: org/elasticsearch/bootstrap/Elasticsearch :
Unsupported major.minor version 52.0
    at java.lang.ClassLoader.defineClass1(Native Method)
    at java.lang.ClassLoader.defineClass(ClassLoader.java:803)
    ...
    at java.lang.ClassLoader.loadClass(ClassLoader.java:358)
    at sun.launcher.LauncherHelper.checkAndLoadMain(LauncherHelper.java:482)
```

This is because the default java version on Debian Jessie is Java 7. Elasticsearch need Java 8:

```shell
    $ sudo apt-get install openjdk-7-jre
    $ sudo update-alternatives --set java /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java
    $ java -version
    openjdk version "1.8.0_111"
```

Running manually
----------------

Elasticsearch can be started using systemd (or classic sysv init scripts), but I like to do things manually :)

```shell
$ mkdir -p es/conf
$ sudo cat /etc/elasticsearch/log4j2.properties > es/conf/log4j2.properties
$ /usr/share/elasticsearch/bin/elasticsearch  --verbose -p es/pid \
    -Edefault.path.logs=es/logs  -Edefault.path.data=es/data  -Edefault.path.conf=es/conf
$ curl 0:9200
{
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "5.0.1",
    ...
  },
  ...
  "tagline" : "You Know, for Search"
}
```

For some intro to Elasticsearch continue with [Guide: Findihg your feet](https://www.elastic.co/guide/en/elasticsearch/guide/current/_finding_your_feet.html) ;)

