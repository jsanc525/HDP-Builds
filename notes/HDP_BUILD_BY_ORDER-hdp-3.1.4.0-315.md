---
title: Build by Order for HDP
author: Joe Azerof
adapted_from: Gonzalo Etse
date: 04-01-24
last_update: 08-01-24
---

# Build by Order

## TDP

### Notes
- in some cases a custom docker image: pierrotws/jenkins-tdp-builder:xenial is needed for builds
- create and rename a copy of the bin/start-build-env.sh file (it's needed for a couple components)
- Modify the file:
    - insert at line 6: docker pull pierrotws/jenkins-tdp-builder:xenial
    - replace tdp-builder at line 21: pierrotws/jenkins-tdp-builder:xenial

## Zookeeper
- original commit: 06cac3e3e20e698d651ac460c80fef7f75603b89
- fixed commit: 2da4bf051a2207cca943c767ba7258ec6264b4b5

### Install
- mvn install:install-file -Dfile=zookeeper-{zkversion}.{hdpversion}.jar -DgroupId=org.apache.zookeeper -DartifactId=zookeeper -Dversion={zkversion}.{hdpversion} -Dpackaging=jar
- mvn install:install-file -Dfile=zookeeper-{zkversion}.{hdpversion}-test.jar -DgroupId=org.apache.zookeeper -DartifactId=zookeeper -Dversion={zkversion}.{hdpversion} -Dpackaging=jar -Dclassifier=tests

### Notes
- uses pierre builder
- multipld hdp tags are grouped together for this build of zookeeper. The same version of zookeeper will work with both hadoop builds needed below
- update build.xml:
   - line 114 old: value="http://repo2.maven.org/maven2/org/apache/ivy/ivy" />
   - line 114 new: value="https://repo1.maven.org/maven2/org/apache/ivy/ivy" />
   - line 122 old: <property name="mvnrepo" value="http://repo2.maven.org/maven2"/>
   - line 122 new: <property name="mvnrepo" value="https://repo1.maven.org/maven2"/>

- update ivysettings.xml:
   - line 21 old: value="http://repo1.maven.org/maven2/" override="false"/>
   - line 21 new: value="https://repo1.maven.org/maven2/" override="false"/>

## Bigdata-interop

- original commit: b4f3b3b89e2eeb72d47fb9682cb0f2d2eef7bdec
- fixed commit: a77484fdacc34b25c33ad1d31307b9e7e3b70bad

### Notes
- update pom.xml:
    - line 109 old: <hadoop.three.version>3.1.1.3.1.4.0-314</hadoop.three.version>
    - line 109 new: <hadoop.three.version>3.1.1.3.1.4.0-315</hadoop.three.version>
    - line 110 old: <junit.version>4.13-SNAPSHOT</junit.version>
    - line 110 new: <junit.version>4.13</junit.version>

## Hadoop
- hdp-2.6.5-292
    - commit: 3091053c59a62c82d82c9f778c48bde5ef0a89a1
- hdp-3.1.4.0-315
    - commit: 64f337236a3b8b25a9f8b77bc8562de22f5429ae

### Install
- hdp-2.6.5-292
- hdp-3.1.4.0-315
    - **gcs-connector (file from client project):** mvn install:install-file -Dfile=gcs-connector-1.9.10.3.1.4.0-315-shaded.jar -DgroupId=com.google.cloud.bigdataoss -DartifactId=gcs-connector -Dversion=1.9.10.3.1.4.0-315 -Dpackaging=jar -Dclassifier=shaded -DgeneratePom=true

### Build
- mvn clean install -Pdist -Dtar -Pnative -DskipTests -Dmaven.javadoc.skip=true

### Notes
- hdp-2.6.5-292
    - required for hive build (shims 0.20)
    - update pom.xml:
        - line 116 old: <jetty.version>6.1.26.hwx</jetty.version>
        - line 116 new: <jetty.version>6.1.26</jetty.version>
- hdp-3.1.4.0-315    
    - com.google.cloud.bigdataoss:gcs-connector:jar:shaded:1.9.10.3.1.4.0-315 Artifact used from clients folder. artifact can be built with [bigdata-interop release repository](https://github.com/hdpmirrors/bigdata-interop-release)


## Tez
- commit: 2be004d2fd89132643d8b9dda21cb1ee0a24c5a9

### Build
- mvn clean install -pl \!tez-ui -Phadoop28 -P\!hadoop27 -DskipTests

## Kafka
- original commit: e1bc65efdca429882ac8e626b6e2d84287b08a3f
- fixed commit: 086894ec3cbd49ddd614947a06f759abef2cfdef

## Notes
- update gradle.properties:
    - line 26 old: repoUrl=http://172.27.44.213/nexus/content/groups/public
    - line 26 new: repoUrl=https://repo1.maven.org/maven2/
    - line 27 old: mavenUrl=http://nexus-private.hortonworks.com:8081/nexus/content/repositories/IN-QA
    -line 27 new: mavenUrl=https://repo1.maven.org/maven2/

### Notes
- requires pierre builder and it is recomnded to change gradle version to 4.5.1
- gradle version management most easily achieved using [SDKMAN!](https://sdkman.io/install)
    - [Usage instructions](https://sdkman.io/usage#use)
    - [Stack-overflow](https://stackoverflow.com/questions/47884051/install-specific-version-of-gradle) with special attention the following instruction
    >1 Great answer! I would add that the gradle binary is added to $HOME/.sdkman/candidates/gradle/[version]/bin. Add this to your PATH environment variable, source the file and you're good to go. – anegru

### Build
- See README in repo for multiple steps

## Arrow
- commit: 65b2db28362a1097382daa023920ad62220f8b9c

### Build
- cd java -mvn clean install -Dskiptests

## Spark2
- original commit: 45f7b124532cceff9257dff3697e494999bbb4ce
- fixed commit: 5ca520afc983df5334b2bdf17857c820f1db01d7

### Notes
- reqs:
    - hive-exec-1.21.2.3.1.4.0-315 (need to find prebuilt)
    - hive-metastore-1.21.2.3.1.4.0-315 (need to find prebuilt)
    - spark-sql_2.11-2.3.2.3.1.4.0-31 (need to find prebuilt)
    - arrow-vector-0.8.0.3.1.4.0-315
    - Kafka
    
### Install
- **hive-exec:** mvn install:install-file -Dfile=hive-exec-1.21.2.3.1.4.0-315.jar -DgroupId=org.spark-project.hive -DartifactId=hive-exec -Dversion=1.21.2.3.1.4.0-315 -Dpackaging=jar
- **hive-metastore:** mvn install:install-file -Dfile=hive-metastore-1.21.2.3.1.4.0-315.jar -DgroupId=org.spark-project.hive -DartifactId=hive-metastore -Dversion=1.21.2.3.1.4.0-315 -Dpackaging=jar
- **kafka-clients:** mvn install:install-file -Dfile=kafka-clients-2.0.0.3.1.4.0-315.jar -DgroupId=org.apache.kafka -DartifactId=kafka-clients -Dversion=2.0.0.3.1.4.0-315 -Dpackaging=jar
- **kafka_2.11:**  mvn install:install-file -Dfile=kafka_2.11-2.0.0.3.1.4.0-315.jar -DgroupId=org.apache.kafka -DartifactId=kafka_2.11 -Dversion=2.0.0.3.1.4.0-315 -Dpackaging=jar
- **spark-sql_2.11:** mvn install:install-file -Dfile=spark-sql_2.11-2.3.2.3.1.4.0-315.jar -DgroupId=org.apache.spark -DartifactId=spark-sql_2.11 -Dversion=2.11-2.3.2.3.1.4.0-315 -Dpackaging=jar

**The following 3 files also require updating kafka-0.10 pom and kafka-0.10-sql pom:**
- **zkclient:** mvn install:install-file -Dfile=zkclient-0.10.jar -DgroupId=org.apache.zookeeper -DartifactId=zkclient -Dversion=0.10 -Dpackaging=jar
- **scala-logging_2.11:** mvn install:install-file -Dfile=scala-logging_2.11-3.9.0.jar -DgroupId=com.typesafe -DartifactId=scala-logging_2.12 -Dversion=3.9.0 -Dpackaging=jar
- **metrics-core:** mvn install:install-file -Dfile=metrics-core-2.2.0.jar -DgroupId=org.apache.kafka -DartifactId=metrics-core -Dversion=2.2.0 -Dpackaging=jar -DgeneratePom=true

### Build
- build/mvn clean install -DskipTests

## Orc
- commit: ff6a21ca519e787cc9f6b2d03ae79185e3e5808c
- required for Hive

### Notes
- requires org.apache.hive:hive-storage-api:jar:2.3.0.3.1.4.0-315. Currently must find built version

### Install
- **hive-storage-api:** mvn install:install-file -Dfile=hive-storage-api-2.3.0.3.1.4.0-315.jar -DgroupId=org.apache.hive -DartifactId=hive-storage-api -Dversion=2.3.0.3.1.4.0-315 -Dpackaging=jar

### Build
- cd java mvn clean install -DskipTest

## Pig
- commit: 771cf70094ea27ec47a20699e6617963e80615af
- required for Parquet
- build paused due to missing dependencies

### Notes
- update build.xml:
    - line 271 old: <property name="mvnrepo" value="http://repo2.maven.org/maven2"/>
    - line 271 new:  <property name="mvnrepo" value="https://repo1.maven.org/maven2"/>

## Parquet
- commit: 077b895ff9776977c9d245f278ed0e117b388c00
- required for hive
- requires Pig
- build paused due to missing dependencies

## Hbase
- original commit: a4745b1cb015baf68e92f3a5b50da25b69aebfb8
- fixed commit: fbc4a40d4c176236a9ea56758146b0eec14513ad

### Notes
- main pom updated at line 1368:
    - from <spark.version>2.3.2.3.1.4.0-314</spark.version>
    - to <spark.version>2.3.2.3.1.4.0-315</spark.version>

### Build
- mvn clean install -DskipTests -Denforcer.skip=true

### Notes

### Build

## Hive
- commit: c0f9f621f56dd1c2687fc7f0f5b0eebab65f0138

### Notes
- requires org.apache.hive:hive-storage-api:jar:2.3.0.3.1.4.0-315. Must currently find an already built version
- requires parquet-hadoop-bundle-1.10.0.3.1.4.0-315.jar. Currently using prebuilt version (see notes in parquet and pig)
- requires Orc, Arrow

### Build
- build: mvn clean install -Pdist -DskipTests -Denforcer.skip=true
