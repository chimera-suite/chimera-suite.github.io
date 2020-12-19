
## Chimera

Chimera is a software suite that aims at better connecting the big data world with semantic technologies. It provides two components for enabling Knowledge Graphs empowered analytics to scale to big data technologies:

- [__OntopSpark__](https://github.com/chimera-suite/OntopSpark): an extension of the  Ontop Ontology Based Data Access (OBDA) system, which uses Apache Spark as a query processing engine for accessing the data stored in data lakes. The integration of a distributed data processing engine such as Apache Spark allows exploiting the Ontop data integration capabilities at its maximum potential, as it brings all the advantages of velocity and parallel computation typical of a distributed system to the task of solving a SPARQL query.

- [__PySPARQL__](https://github.com/chimera-suite/PySPARQL): a library that allows the users to query a SPARQL endpoint using a python notebook, process the response inside Apache Spark, and eventually store the Spark DataFrame/GraphFrame into the data lake.

## Scope

This project aims to build a __general framework__ that can be possibly applied to many industrial scenarios, in order  to improve the support of KG-empowered analytical solutions to big data and make possible the creation of _round-tripping_ data pipelines. Therefore, we have developed Chimera for being __scalable__ and __problem-agnostic__, to encourage the adoption by many companies.

Hence, we think it is very important to make all components available as [Docker images](https://hub.docker.com/u/chimerasuite). We have also created an [infrastructure template](https://github.com/chimera-suite/infrastructure) and a [demo](https://github.com/chimera-suite/use-case) for showing how to use Chimera in practice for performing a _round-trip_ analysis.

## Rond-tripping analyses

The two components of Chimera, namely OntopSpark and PySPARQL, can be used separately one from the other. However, the synergy between the two enables _round-tripping_ pipelines, where data coming back and forth from Spark are continuously enriched with semantic technologies. An example of _round tripping_ concept is expressed in the following figure.

<img src="chimera_infrastructure.png">

In the architectural example, we can see that the Data Scientists can write analytical SPARQL queries on notebooks using PySPARQL. Those queries are sent to Jena Fuseki, which resolves the part inside the `SERVICE` clause (known as _federated query_) using OntopSpark, by performing Ontology Based Data Access of the data stored in the HDFS data lake using Spark as query processing engine, and translating the SQL responses into RDF triples by using the R2RML mappings and the DB-descriptive ontology. Once the triples are back from  OntopSpark, Jena Fuseki has to enrich them by using the internal Knowledge Graph and send back the results to the notebook. At this point, the result is available to the user in the form of Spark DataFrame or GraphFrame, which can be further analyzed using the notebook and, if necessary, persisted in the data lake by executing a PySPARQL function.  The materialization task ends the _round-trip_ circle. A new analysis iteration can be started if needed.

## Configuration

### 1. OntopSpark

__PREREQUISITE__: For running OntopSpark it is needed to have installed `docker-compose` on your local system. If you do not have it installed, please follow this [official guide](https://docs.docker.com/compose/install/).

It is possible to run the Ontop endpoint by simply running the [`docker-compose-ontop.yml`](https://github.com/chimera-suite/infrastructure/blob/main/docker-compose-ontop.yml) file. Before starting, this file has to be configured with the following parameters:

```
environment:
  - `ONTOP_ONTOLOGY_FILE`=/opt/ontop/input/DB-ontology.owl"  # TODO
  - `ONTOP_MAPPING_FILE`=/opt/ontop/input/mapping.obda"  #TODO
  - `ONTOP_PROPERTIES_FILE`=/opt/ontop/input/jdbc.properties"  #TODO
 volumes:
    - "./ontop/input:/opt/ontop/input"   # load Ontop configuration files inside docker environment
    - "./ontop/jdbc:/opt/ontop/jdbc"   # load Spark JDBC driver inside docker environment
```

Where the `environment` section contains the Ontop's variables for instantiating the endpoint, respectively:
1. `ONTOP_ONTOLOGY_FILE` : the file containing the ontological concepts needed by the Ontop reasoner for describing the semantic of the relational data stored in the data lake.
3. `ONTOP_MAPPING_FILE` : the mapping file for the RDF-to-SQL translations performed by the Virtual Knowledge Graph mechanism of Ontop.
2. `ONTOP_PROPERTIES_FILE` : a configuration file for correctly instantiating the JDBC connection to the Apache Spark query engine. In particular, the file must contain the JDBC address of the Apache Spark Thriftserver.

Instead, the `volumes section` loads the Ontop's files inside the docker environment. By default (in the code above), you need to create two folders, `/ontop/input` and `/ontop/JDBC` for storing respectively the configuration files (`.owl`,`.obda`,`.properties`) and the JDBC `.jar` driver file.


 Now, it is possible to start the Ontop instance by running the following command:

```
docker-compose -f docker-compose-ontop.yml up -d
```
The web interface is available at [http://ontop:8080](http://localhost:8090).

You can learn how to use OntopSpark inside an infrastructure by following this [demo](https://github.com/chimera-suite/use-case).


### 2. PySPARQL

It is possible to install the PySPARQL library on a python notebooks by simply running the following command.

```
!pip install SPARQL2Spark
```

For more details on how to use the library, please see the [PySPARQL documentation](https://pypi.org/project/PySPARQL/0.0.5/).

## Aknowledgements

The Chimera project has been developed and maintained for the next years thanks to Ricerca sul Sistema Energetico S.p.A. co-founding and Politecnico di Milano’s own resources.
