---
title: Build by Order for HDP
author: Joe Azerof
adapted_from: Gonzalo Etse
date: 04-01-24
last_update: 05-01-24
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
- clean commit: 06cac3e3e20e698d651ac460c80fef7f75603b89
- fixed commit: 2da4bf051a2207cca943c767ba7258ec6264b4b5

### Install
- mvn install:install-file -Dfile=zookeeper-{zkversion}.{hdpversion}.jar -DgroupId=org.apache.zookeeper -DartifactId=zookeeper -Dversion={zkversion}.{hdpversion} -Dpackaging=jar
- mvn install:install-file -Dfile=zookeeper-{zkversion}.{hdpversion}-test.jar -DgroupId=org.apache.zookeeper -DartifactId=zookeeper -Dversion={zkversion}.{hdpversion} -Dpackaging=jar -Dclassifier=tests

### Notes
- uses pierre builder
- update build.xml:
   - line 114 old: value="http://repo2.maven.org/maven2/org/apache/ivy/ivy" />
   - line 114 new: value="https://repo1.maven.org/maven2/org/apache/ivy/ivy" />
   - line 122 old: <property name="mvnrepo" value="http://repo2.maven.org/maven2"/>
   - line 122 new: <property name="mvnrepo" value="https://repo1.maven.org/maven2"/>

- update ivysettings.xml:
   - line 21 old: value="http://repo1.maven.org/maven2/" override="false"/>
   - line 21 new: value="https://repo1.maven.org/maven2/" override="false"/>

## Bigdata-interop
- clean commit: b4f3b3b89e2eeb72d47fb9682cb0f2d2eef7bdec
- fixed commit: a77484fdacc34b25c33ad1d31307b9e7e3b70bad

### Notes
- update pom.xml:
    - line 109 old: <hadoop.three.version>3.1.1.3.1.4.0-314</hadoop.three.version>
    - line 109 new: <hadoop.three.version>3.1.1.3.1.4.0-315</hadoop.three.version>
    - line 110 old: <junit.version>4.13-SNAPSHOT</junit.version>
    - line 110 new: <junit.version>4.13</junit.version>

## Hadoop
- commit: 64f337236a3b8b25a9f8b77bc8562de22f5429ae

### Install
- **gcs-connector (file from client project):** mvn install:install-file -Dfile=gcs-connector-1.9.10.3.1.4.0-315-shaded.jar -DgroupId=com.google.cloud.bigdataoss -DartifactId=gcs-connector -Dversion=1.9.10.3.1.4.0-315 -Dpackaging=jar -Dclassifier=shaded -DgeneratePom=true

### Build
- mvn clean install -Pdist -Dtar -Pnative -DskipTests -Dmaven.javadoc.skip=true

### Notes
- com.google.cloud.bigdataoss:gcs-connector:jar:shaded:1.9.10.3.1.4.0-315 Artifact used from clients folder. artifact can be built with [bigdata-interop release repository](https://github.com/hdpmirrors/bigdata-interop-release)

## Tez
- commit: 2be004d2fd89132643d8b9dda21cb1ee0a24c5a9

### Build
- mvn clean install -pl \!tez-ui -Phadoop28 -P\!hadoop27 -DskipTests

## Kafka
- clean commit: e1bc65efdca429882ac8e626b6e2d84287b08a3f
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

## Spark2
- commit: 45f7b124532cceff9257dff3697e494999bbb4ce

### Notes
- reqs:
    - hive-exec-1.21.2.3.1.4.0-315 (from client folder)
    - hive-metastore-1.21.2.3.1.4.0-315 (from client folder)
    - arrow-vector-0.8.0.3.1.4.0-315 (build?)
    - Kafka
    
### Install
- **hive-exec:** mvn install:install-file -Dfile=hive-exec-1.21.2.3.1.4.0-315.jar -DgroupId=org.spark-project.hive -DartifactId=hive-exec -Dversion=1.21.2.3.1.4.0-315 -Dpackaging=jar
- **hive-metastore:** mvn install:install-file -Dfile=hive-metastore-1.21.2.3.1.4.0-315.jar -DgroupId=org.spark-project.hive -DartifactId=hive-metastore -Dversion=1.21.2.3.1.4.0-315 -Dpackaging=jar
- kafka-clients:
- mvn install:install-file -Dfile=kafka-clients-2.0.0.3.1.4.0-315.jar -DgroupId=org.apache.kafka -DartifactId=kafka-clients -Dversion=2.0.0.3.1.4.0-315 -Dpackaging=jar
- kafka_2.11:  mvn install:install-file -Dfile=kafka_2.11-2.0.0.3.1.4.0-315.jar -DgroupId=org.apache.kafka -DartifactId=kafka_2.11 -Dversion=2.0.0.3.1.4.0-315 -Dpackaging=jar
The following 3 files also require updating kafka-0.10 pom and kafka-0.10-sql pom:
- org.I0Itec: mvn install:install-file -Dfile=zkclient-0.10.jar -DgroupId=org.I0Itec -DartifactId=zkclient_2.11 -Dversion=0.10 -Dpackaging=jar
- com.typesafe: mvn install:install-file -Dfile=scala-logging_2.11-3.9.0.jar -DgroupId=com.typesafe -DartifactId=scala-logging_2.12 -Dversion=3.9.0 -Dpackaging=jar
- com.yammer: mvn install:install-file -Dfile=metrics-core-2.2.0.jar -DgroupId=com.yammer -DartifactId=metrics-core -Dversion=2.2.0 -Dpackaging=jar -DgeneratePom=true

### Build
- ./dev/make-distribution.sh --name tdp --tgz -Phive -Phive-thriftserver -Pyarn -Phadoop-3.1 -Pflume
