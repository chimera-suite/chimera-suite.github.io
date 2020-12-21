
## Chimera

Chimera is a software suite that aims at better connecting the big data world with semantic technologies. It provides two components for enabling Knowledge Graphs empowered analytics scalable to big data technologies:

- [__OntopSpark__](https://github.com/chimera-suite/OntopSpark): an extension of the Ontop Ontology Based Data Access (OBDA) system, which uses Apache Spark as a query processing engine for accessing the data stored in data lakes. The integration of a distributed data processing engine such as Apache Spark allows exploiting the Ontop data integration capabilities at its maximum potential, as it brings all the advantages of velocity and parallel computation typical of a distributed system to the task of solving a SPARQL query.

- [__PySPARQL__](https://github.com/chimera-suite/PySPARQL): a library that allows the users to query a SPARQL endpoint using a python notebook, process the response with Apache Spark, and eventually store the Spark DataFrame / GraphFrame into the data lake.

## Scope

This project aims to build a __general framework__ that can be possibly applied to many industrial scenarios, in order  to improve the support of KG-empowered analytical solutions to big data and make possible the creation of _round-tripping_ data pipelines. Therefore, we have developed Chimera for being __scalable__ and __problem-agnostic__, to encourage the adoption by many companies.

Hence, we think it is very important to make all components available as [__Docker images__](https://hub.docker.com/u/chimerasuite). We have also created an [infrastructure template](https://github.com/chimera-suite/infrastructure) and a [__demo__](https://github.com/chimera-suite/use-case) for showing how to use Chimera in practice for performing a _round-trip_ analysis.

## Features

### Ontology Based Data Access (OBDA)

The OBDA paradigm enables creating analytical SPARQL queries based on data physically stored in relational formats by writing queries over an ontology that makes transparent the task of retrieving data from several SQL tables and joining them. Among the reference OBDA systems, we decided to develop an extension for [Ontop](https://ontop-vkg.org/), as it is one of the first to be offered as a commercial solution. We developed the [__OntopSpark__](https://github.com/chimera-suite/OntopSpark) extension for enabling Ontop to perform OBDA by querying relational data physically stored in Spark tables. OntopSpark uses the Ontop's Virtual Knowledge Graph (VKG) mechanism, which allows creating RDF representations of relational data without allocating additional space.

OntopSpark needs three files to respond to SPARQL queries by building the corresponding VKGs: 1. a __DB-descriptive ontology__ file (usually .owl or .ttl) containing the ontological concepts in OWL2QL profile needed by the Ontop reasoner the semantic of the relational data stored in the Spark tables, 2. a configuration file for correctly instantiating a JDBC connection to the database, and 3. a __mapping__ file for the RDF-to-SQL translation of the VKGs. For more details on how to configure OntopSpark, see the [configuration section](#1-ontopspark).

OntopSpark is available in two different packages, namely _OntopSpark-Protégé_ and _OntopSpark-CLI_. The first one is an extension that allows building ontologies and mappings using the graphical interface of Protégé, while the second exposes a SPARQL endpoint (web GUI or CLI) used for industrial deployment. We use the _OntopSpark-CLI_ package for building a [docker image](https://hub.docker.com/r/chimerasuite/ontop) available on DockerHub.

### SPARQL queries using notebooks and Apache Spark

The [__PySPARQL__](https://github.com/chimera-suite/PySPARQL) python module allows the users to query a SPARQL endpoint using a python notebook and to process the response inside Apache Spark. PySPARQL leverages pyspark to manage Spark DataFrames, and uses well known libraries such as SPARQLWrapper and rdflib to handle the communication with a SPARQL endpoint and manage the result. PySPARQL is tested with multiple Spark versions and is available on [__PyPi__](https://pypi.org/project/PySPARQL/).

The library retrieves the results and materializes them inside the configured Spark Session. Users shall specify the endpoint configuration at initialization time and change it during the program execution. In particular, the output type directly depends on the SPARQL query type. __SELECT__ queries return _Spark DataFrames_ in which the columns directly correspond to the variables declared in the SPARQL query. However, PySPARQL does not covert the value types. The users can then process the DataFrame inside Spark and, if necessary, save the DataFrame as a Spark table. __CONSTRUCT__ queries return either a _DataFrame_ or a _GraphFrame_ depending on what the user chooses to materialize. In both cases the data resemble the constructed graph.

### Rond-tripping analysis

The two components of Chimera, namely OntopSpark and PySPARQL, can be used separately one from the other. However, the synergy between the two enables _round-tripping_ pipelines, where data coming back and forth from Spark are continuously enriched with semantic technologies. An example of _round tripping_ concept is expressed in the following figure.

<img src="chimera_infrastructure.png">

In the architectural example, we can see that the Data Scientists can write analytical SPARQL queries on notebooks using PySPARQL. Those queries are sent to Jena Fuseki, which resolves the part inside the `SERVICE` clause (known as _federated query_) using OntopSpark, by performing Ontology Based Data Access of the data stored in the HDFS data lake using Spark as query processing engine, and translating the SQL responses into RDF triples by using the R2RML mappings and the DB-descriptive ontology. Once the triples are back from  OntopSpark, Jena Fuseki has to enrich them by using the internal Knowledge Graph and send back the results to the notebook. At this point, the result is available to the user in the form of Spark DataFrame or GraphFrame, which can be further analyzed using the notebook and, if necessary, persisted in the data lake by executing a PySPARQL function.  The materialization task ends the _round-trip_ circle. A new analysis iteration can be started if needed.

## Configuration

### 1. OntopSpark

__PREREQUISITE__: For running OntopSpark it is needed to have installed `docker-compose` on your local system. If you do not have it installed, please follow this [official guide](https://docs.docker.com/compose/install/).

It is possible to run the Ontop endpoint by simply adding the following configuration in a `docker-compose.yml` file.

```
ontopSpark:
  hostname: ontopSpark
  container_name: ontopSpark
  image: chimerasuite/ontop:fd38fec7a0
  environment:
    - `ONTOP_ONTOLOGY_FILE`=/opt/ontop/input/DB-ontology.owl"  # TODO
    - `ONTOP_MAPPING_FILE`=/opt/ontop/input/mapping.obda"  #TODO
    - `ONTOP_PROPERTIES_FILE`=/opt/ontop/input/jdbc.properties"  #TODO
  ports:
    - "8090:8080"     # 8090 is the port outside the docker-compose virtual network
  volumes:
    - "./ontop/input:/opt/ontop/input"   # load Ontop configuration files inside docker environment
    - "./ontop/jdbc:/opt/ontop/jdbc"   # load Spark JDBC driver inside docker environment
```

Where the `environment` section contains the Ontop's variables for instantiating the endpoint, respectively:
1. `ONTOP_ONTOLOGY_FILE` : the file containing the ontological concepts needed by the Ontop reasoner for describing the semantic of the relational data stored in the data lake.
3. `ONTOP_MAPPING_FILE` : the mapping file for the RDF-to-SQL translations performed by the Virtual Knowledge Graph mechanism of Ontop.
2. `ONTOP_PROPERTIES_FILE` : a configuration file for correctly instantiating the JDBC connection to the Apache Spark query engine. In particular, the file must contain the JDBC address of the Apache Spark Thriftserver.

Instead, the `volumes section` loads the Ontop's files inside the docker environment. By default (in the code above), you need to create two folders, `/ontop/input` and `/ontop/JDBC` for storing respectively the configuration files (`.owl`,`.obda`,`.properties`) and the JDBC `.jar` driver file.

After setting the appropriate configuration, it is possible to start the Ontop instance by excuting the following terminal command:

```
docker-compose up -d
```
The web interface is available at [http://localhost:8090](http://localhost:8090) (if docker is running on the same pc used for accessing the web UI). You can use this web interface to write SPARQL queries on OntopSpark or use a _federated query_ approach as shown in the PySPARQL configuration section.

For learning how to use OntopSpark inside a data analysis pipeline, you can follow this [__demo__](https://github.com/chimera-suite/use-case).


### 2. PySPARQL

It is possible to install the PySPARQL library on a python notebook by simply running the following command.

```
!pip install SPARQL2Spark
```

For executing queries inside a python notebook, it is possible to use the following code snippet. The query part inside the __SERVICE__ clause, known as _federated query_, is resolved by OntopSpark using the OBDA approach, which retrieves the data from Spark tables and translates the SQLresponses into RDF triples. Note that the _federated query_ not only works with OntopSpark, but also with many other SPARQL endpoints.

```
from pyspark import SparkContext, SparkConf
from pyspark.sql import SparkSession, HiveContext

# Instantiate a Spark session
spark = (SparkSession.builder                     # TODO: set Spark session
          .appName("𝙢𝙮-𝙖𝙥𝙥-𝙣𝙖𝙢𝙚")
          .config("hive.metastore.uris", "𝙢𝙚𝙩𝙖𝙨𝙩𝙤𝙧𝙚-𝙖𝙙𝙙𝙧𝙚𝙨𝙨")
          .enableHiveSupport()
          .getOrCreate())

# Import PySPARQL library
from PySPARQL.Wrapper import PySPARQLWrapper

sparql_endpoint = "𝙮𝙤𝙪𝙧-𝙨𝙥𝙖𝙧𝙦𝙡-𝙚𝙣𝙙𝙥𝙤𝙞𝙣𝙩"          # TODO: set endpoint
query = """PREFIX ....                            # TODO: write SPARQL query
           SELECT ....
           WHERE {
              ......
              SERVICE <𝙊𝙣𝙩𝙤𝙥𝙎𝙥𝙖𝙧𝙠 𝙖𝙙𝙙𝙧𝙚𝙨𝙨> { ... 𝙊𝘽𝘿𝘼_𝙦𝙪𝙚𝙧𝙮 ... }
              ....
           }
        """

wrapper = PySPARQLWrapper(spark, sparql_endpoint)
result = wrapper.query(query)  # execute the query
resultDF = result.dataFrame    # tanslate the restult into a Spark DataFrame
```

As seen above, PySPARQL can convert SPARQL query results in Spark DataFrames / GraphFrames. Furthermore, using the following code to persist the DataFrames / GraphFrames in Spark tables physically stored in a data lake.

```
resultDF.write.mode("𝙬𝙧𝙞𝙩𝙚-𝙢𝙤𝙙𝙚").saveAsTable('𝙎𝙥𝙖𝙧𝙠-𝙩𝙖𝙗𝙡𝙚-𝙣𝙖𝙢𝙚')  # TODO : set parameters
```

To learn how to use PySPARQL inside a Jupyter notebook, you can see two notebook examples available in this [__demo__](https://github.com/chimera-suite/use-case).

## Development and Maintenance

The Chimera project has been developed by Politecnico di Milano. Thanks to Ricerca sul Sistema Energetico S.p.A. co-founding and Politecnico di Milano's resources, Chimera will be maintained for the next years and updated whenever a new version of Ontop or Apache Spark becomes available.
