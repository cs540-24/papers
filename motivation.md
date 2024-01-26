Motivation
==========


Cloud technologies and the monetization of user tracking have
brought big data in an easy-to-access form upon us, but the tools,
practices, and general understanding on how to deal with it are
lacking. In particular, such data are operational and traditional
statistical and machine learning techniques are not
appropriate. Such data are large and we can't expect to analyze it
on our laptop using our favorite tool or language, but need to build
software to do that. 

Hence this course: or how do we move from software engineering and
data analysis to evidence engineering.

The technology is key when dealing with big data
-------------------------------------------------

Virtual cloud via docker containers
   1. Allows to access more resources transparently for a single
     application.
   1. Much more general that hadoop or spark
   1. Allows mixing various technologies on a single host or among
      hosts

Spark
   1. Automates breaking data into pieces
   1. Needs to store data in hdfs first: slow, tedious
   1. Can easily overwhelm the computation (e.g., collect)
   1. Performance issues


OpenMPI
   1. Allows larger RAM by sharing across hosts
   1. Quite difficult to use

Graph databases:
   1. Make relationship queries fast and convenient
   1. A lot of effort preparing and inputing data
   1. Immature, performance problems

Graph libraries: Boost
   1. Best performance
   1. Some routines compatible with OpenMPI
   1. Need to prepare data
   
Basic unix commands:
   1. sort - unbeatable performance with right parameters
   1. Flexible
   1. Enhance with awk/perl/python for associative arrays (has
      tables)

Text analysis:
   1. Forget LSI/LDA: doc2vec/word2vec rule


SQL databases:
   1. Don't work for large data


The practices are totally different in big data
-----------------------------------------------

Understanding the limitations and use cases for various technologies
is a must.

* Planning data workflow
    1. Tradoffs among CPU/RAM/disk/network
    1. Processing steps
    1. Debugging steps (need to plan ahead)
    1. Various approaches to prototyping
    1. Resilience

* Establishing data quality issues
    1. Identity of actors
    1. Digital signature of entities: file content, filenames, text
     messages, ... 

* Debugging noisy graphs
    

Applications as Aims
--------------------

Unlike with small data, building a full application within
one semester is a dream. A prototype, perhaps...


Many things need to be tried in parallel:
   1. Can necessary data be processed
   1. Would it be useful for an application (need to evaluate on a
      subset)
   1. How debugging will be conducted?


Example motivations for applications based on very large data of
all public version control systems.


1. Relationship among developers based on common artifacts modified
    1. Different from traditional social metrics based on communication
    1. Can help find relationship among actors
	1. Can help link multiple identities of a single actor


1. Relationship among developers based on similar language
    1. Can find developers working on similar topics or projects
	1. Can help link multiple identities of a single actor

1. Finding suitable recruits for an OSS project
    1. Use the relationships based on code to identify communities
       and use them in conjunction with developer performance
       to make recommendations.

1. Orphanages: groups of no longer maintained projects
    1. To find maintainers before the owner leaves
	1. To identify alternatives in the supply chain
	1. To provide leads to volunteers willing to help



