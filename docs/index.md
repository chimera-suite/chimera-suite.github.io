## Chimera

Chimera is a software suite that aims at better connecting the big data world with semantic technologies. It provides two components for enabling Knowledge Graphs empowered analytics to scale to big data technologies:

- [__OntopSpark__](https://github.com/chimera-suite/OntopSpark):
en extension that enables Ontop to perform Ontology Based Data Access (OBDA) of relational data using Apache Spark. Given as input the bindings between RDF statements andSparkSQL queries, it is possible to submit SPARQL queries to the Ontop endpoint and obtain a response  based  on  big  volumes  of  relational  data  processed by Spark and physically stored in a data lake. The integration of a distributed data processing engine such as Apache Spark allows exploiting the Ontop data integration capabilities at its maximum potential, because it brings all the advantages of velocity and parallel computation typical of a distributed system to the task of solving a SPARQL query.

- [__PySPARQL__](https://github.com/chimera-suite/PySPARQL): a library that allows the users to query a SPARQL endpoint using a python notebook, process the response inside Apache Spark, and eventually store the Spark DataFrame/GraphFrame into the data lake.

## Rond-tripping analyses

The two components of chimera, namely OntopSpark an PySPARQL, can  be  used  separately  one  from  the  other. However, is the  synergy  between  the  two that enables _round-tripping_ pipelines, where data coming back and forth from Spark are continuously enriched with semantic technologies. An example of _round tripping_ concept is expressed in following figure.

<img src="chimera_infrastructure.png" height="400px" ></img>

We can see that the Data Scientists writes their analytical SPARQL queries on notebooks using PySPARQL, The query is sent to Jena Fuseki, which resolves the query part inside the `SERVICE` clause, known as _federated query_, using OntopSpark for performing Ontology Based Data Access of the data stored in the HDFS data lake using Spark as query processing engine, and translating the SQL responses into RDF triples by using the R2RML mappings and the DB-descriptive  ontology. Once the  triples  are  back  from  OntopSpark,  Jena  Fuseki has to enrich them by using the internal Knowledge Graph and send back the results to the notebook. At this point, the result is available to the user in form of Spark DataFrame or GraphFrame, that can be further analyzed using the notebook and, if necessary, persisted in the data lake by executing a PySPARQL function.  The  materialization task ends the _round-trip_ circle. A new analysisiteration can be started if needed.

## Infrastructure

The goal of this project is to build a __general framework__ that can be possibly applied to many industrial scenarios, in order to improve the support of KG-empowered analytical solutions to big data and to make possible the creation of _round-tripping_ data pipelines. Therefore, we have developed Chimera for being __scalable__ and __problem-agnostic__, to en-courage the adoption by many companies.

Hence, we think it is very important to make all components available as [Docker images](https://hub.docker.com/orgs/chimerasuite/repositories). We have also created an [infrastructure template](https://github.com/chimera-suite/infrastructure) and a [demo](https://github.com/chimera-suite/use-case) for showing how to use Chimera in practice for perfoming a _round-trip_ analysis.

## Cofiguration

__PREREQUISITE__: For running both the Chimera components it's needed to have installed on your local system `docker-compose`. If you don't have it installed, plese follow this [official guide](https://docs.docker.com/compose/install/).

### 1. OntopSpark

It's possible to run the Ontop enpoint by simply running the `docker-compose-ontop.yml` file available in the [infrastructure template](https://github.com/chimera-suite/infrastructure) with the following command.

```
docker-compose -f docker-compose-ontop.yml up -d
```
This starts an Ontop instance.
In particular the instance is configured to communicate with the Apache Spark Thriftserver.
The web interface is available at [http://ontop:8080](http://localhost:8090).
The docker image can be configured with the following parameters in the `docker-compose-ontop.yml` file.

```
environment:
  - "ONTOP_ONTOLOGY_FILE=/opt/ontop/input/pizza-minimal-ontop.owl"
  - "ONTOP_MAPPING_FILE=/opt/ontop/input/pizza-minimal-ontop.obda"
  - "ONTOP_PROPERTIES_FILE=/opt/ontop/input/pizza-minimal-ontop.properties"
volumes:
    - "./ontop/input:/opt/ontop/input" # load Ontop configuration files inside docker image
    - "./ontop/jdbc:/opt/ontop/jdbc" # load Spark JDBC driver
```

Where the `environment` contains the Ontop environment variables for instantiating the endpoint, respectively:
1. an ontology file (usually .owl or .ttl) containing the ontological concepts needed by the Ontop reasoner
3. a mappingfile for the RDF-to-SQL translation performed by the Virtual Knowledge Graph mechanism of Ontop.
2. a configuration file for correctly instantiating a JDBC connection to the database.

### 2. PySPARQL

It's possible to install the PySPARQL library on your python notebook by simply running the following command.

```
!pip install SPARQL2Spark
```

For more details on how to use the library, please see the [PySPARQL documentation](link_mancante).

## aknowledgements

The Chimera project has been developed by Politecnico di Milano. The project is mantainde thanks to RSE co-founding and Politecnico di Milano.
