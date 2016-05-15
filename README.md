[![Gitter chat](http://img.shields.io/badge/gitter-join%20chat%20%E2%86%92-brightgreen.svg)](https://gitter.im/openzipkin/zipkin) [![Build Status](https://travis-ci.org/openzipkin/zipkin-java.svg?branch=master)](https://travis-ci.org/openzipkin/zipkin-java) [![Download](https://api.bintray.com/packages/openzipkin/maven/zipkin-java/images/download.svg) ](https://bintray.com/openzipkin/maven/zipkin-java/_latestVersion)

# zipkin-java
This project is a native java port of [zipkin](https://github.com/openzipkin/zipkin), which was historically written in scala+finagle. This includes a dependency-free library and a [spring-boot](http://projects.spring.io/spring-boot/) replacement for zipkin's query and collector services. Storage options include in-memory, JDBC (mysql), Cassandra, and Elasticsearch.

## Core Library
The [core library](https://github.com/openzipkin/zipkin-java/tree/master/zipkin/src/main/java/io/zipkin) requires minimum language level 7. While currently only used by the server, we expect this library to be used in native instrumentation as well.

This includes built-in codec for both thrift and json structs. Direct dependencies on thrift or moshi (json library) are avoided by minifying and repackaging classes used. The result is a 256k jar which won't conflict with any library you use.

Ex.
```java
// your instrumentation makes a span
archiver = BinaryAnnotation.create(LOCAL_COMPONENT, "archiver", Endpoint.create("service", 127 << 24 | 1));
span = Span.builder()
    .traceId(1L)
    .name("targz")
    .id(1L)
    .timestamp(epochMicros())
    .duration(durationInMicros)
    .addBinaryAnnotation(archiver);

// Now, you can encode it as json or thrift
bytes = Codec.JSON.writeSpan(span);
bytes = Codec.THRIFT.writeSpan(span);
```

## Storage Component
Zipkin includes a [StorageComponent](https://github.com/openzipkin/zipkin-java/blob/master/zipkin/src/main/java/zipkin/storage/StorageComponent.java), used to store and query spans and dependency links. This is used by the server and those making custom servers, collectors, or span reporters. For this reason, storage components have minimal dependencies; many run on Java 7.

Ex.
```java
// this won't create network connections
storage = CassandraStorage.builder()
                          .contactPoints("my-cassandra-host").build();

// but this will
trace = storage.spanStore().getTrace(traceId);

// clean up any sessions, etc
storage.close();
```

### InMemoryStorage
The [InMemoryStorage](https://github.com/openzipkin/zipkin-java/blob/master/zipkin/src/main/java/zipkin/storage/InMemoryStorage.java) component is packaged in zipkin's core library. It is not persistent, nor viable for realistic work loads. Its purpose is for testing, for example starting a server on your laptop without any database needed.

### JDBCStorage
The [JDBCStorage](https://github.com/openzipkin/zipkin-java/tree/master/zipkin-storage/jdbc) component currently is only tested with MySQL 5.6-7. It is designed to be easy to understand, and get started with. However, it has [known performance issues at larger scales](https://github.com/openzipkin/zipkin-java/issues/233). For example, queries will eventually take seconds to return if you put a lot of data into it.

### CassandraStorage
The [CassandraStorage](https://github.com/openzipkin/zipkin-java/tree/master/zipkin-storage/cassandra) component is tested against Cassandra 2.2+. It is designed for larger scales of data. For example, it has manually implemented indexes to make querying larger data more performant.

### ElasticsearchStorage
The [ElasticsearchStorage](https://github.com/openzipkin/zipkin-java/tree/master/zipkin-storage/elasticsearch) component is tested against Elasticsearch 2.3. It is designed for larger scales of data, and works on json directly. The Elasticsearch component is the newest option.

## Server
The [zipkin server](https://github.com/openzipkin/zipkin-java/tree/master/zipkin-server)
receives spans via HTTP POST and respond to queries from its UI. It can also run collectors, such as Scribe or Kafka.

To run the server from the currently checked out source, enter the following.
```bash
# Build the server and also make its dependencies
$ ./mvnw -DskipTests --also-make -pl zipkin-server clean install
# Run the server
$ java -jar ./zipkin-server/target/zipkin-server-*exec.jar
```

Note that the server requires minimum JRE 8.

## Artifacts
### Library Releases
Releases are uploaded to [Bintray](https://bintray.com/openzipkin/maven/zipkin-java).
### Library Snapshots
Snapshots are uploaded to [JFrog](http://oss.jfrog.org/artifactory/oss-snapshot-local) after commits to master.
### Docker Images
Released versions of zipkin-server are published to Docker Hub as `openzipkin/zipkin-java`.
See [docker-zipkin-java](https://github.com/openzipkin/docker-zipkin-java) for details.
