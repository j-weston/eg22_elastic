# Living the Life Elastic

2022 Evergreen Online Conference

Bill Erickson

Software Development Engineer, King County Library System

[Slides as Markdown / HTML](https://github.com/berick/Presentations/tree/master/Evergreen-2022)

---

# A Report

## Project Review and History

## Elasticsearch In Action at King County

## Day to Day Administration

---

# Project Goal

* Improve Evergreen catalog search speed for staff.

---

# A Brief History

* Evergreen Begins
* Rise of [Solr](https://solr.apache.org/) and discovery layers
* KCLS adopts EG, soon migrates to 3rd-party catalog
* Jeff G presents(?) on Elasticsearch-driven mobile catalog
* Elasticsearch proof-of-concept implementation for EG
* Blake GH opens [LP1844418](https://bugs.launchpad.net/evergreen/+bug/1844418)
* Angular Catalog development proceeds in parallel
* Angular Catalog + Elasticsearch limited staff use at KCLS 2020
* KCLS general use late 2021

---

# What is Elasticsearch?

> Elasticsearch is a distributed, open source search and analytics
> engine for all types of data, including textual, numerical,
> geospatial, structured, and unstructured. Elasticsearch is built on
> Apache Lucene and was first released in 2010 by Elasticsearch N.V.
> (now known as Elastic). Known for its simple REST APIs, distributed
> nature, speed, and scalability...

Source: https://www.elastic.co/what-is/elasticsearch

---

# Why Elasticsearch?

* Similar to Solr
* Ease of use
* Broad feature set
* Excellent Documentation and Examples
* I liked the API
* Industry use outside the library world
* Clustering / Replication
* Open source w/ vendor support/additions

---

# Other Benefits to External Indexing

* Indexing speed
    * KCLS 1.1M records; 3.6M items
    * 4 parallel: 1 hour 45 mins
* Takes heavy search query load off primary PG Database
* Searches report total result count / no estimates
* Opportunities for new types of searches with minimal backend development.
* Parallel, Interchangeable Datasets

---

# What's Implemented?

* An Evergreen API for Keyword, Title, Author, etc. searches
* Some Numeric Searches (e.g. not Item Barcode)
* MARC search
* Query String support

---

# Query String Examples

### Query String supported added to Keyword field

* Give me everything: \*:\*
* Give me the new stuff: pubdate:2020
* Ranges
    * pubdate:[2001 TO 2010]
    * create_date:[2021-01 TO 2021-02]
    * pubdate:>=2020
* Boolean Grouping
    * (kw:dogs AND (pubdate:2021 OR pubdate:2022)) OR (ti:cats AND NOT pubdate:2022)

---

# Analysis and Normalization

* R.E.M.
* REM
* R E M
* Its A Wonderful Life

---

# Local Additions & Modifications

* MARC match option selector
* 'contains exact'  match opt
* MARC regex search
    * .{24}[^6]{3}.{13}
    * [Graphic Novels](https://evgstaging.kcls.org/eg2/en-US/staff/catalog/search?org=1&limit=10&marcTag=008&marcTag=655&marcSubfield=&marcSubfield=a&marcValue=.%7B24%7D%5B%5E6%5D%7B3%7D.%7B13%7D&marcValue=graphic%20novels&matchOp=regexp&matchOp=phrase)

---

# Pending Features

* "Did You Mean" ([Elastic Docs](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/search-suggesters-phrase.html))
* Search Results Highlight ([Elastic Docs](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/search-request-highlighting.html))
* Autosuggest ([Elastic Docs](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/search-suggesters-completion.html))
* Sort by Populatrity
* Copy Location Group filtering
* Org Unit Lasso filtering
* Others?

---

# Setup and Administration

---

# Installation

    !sh
    $ sudo apt install openjdk-11-jre-headless

    $ wget 'https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.8.11.deb'

    $ sudo dpkg -i elasticsearch-6.8.11.deb

    $ sudo systemctl start elasticsearch

    $ sudo systemctl enable elasticsearch

    $ sudo cpan Search::Elasticsearch::Client::6_0

---

# Plugin - International Components for Unicode

[Elasticserach ICU Analysis Plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-icu.html)

    !sh
    $ cd /usr/share/elasticsearch/

    $ sudo bin/elasticsearch-plugin install analysis-icu

---

# Building Indexes

    !sh
    cd /home/opensrf/Evergreen/Open-ILS/src/support-scripts/

    ./elastic-index.pl --index-name kcls-1 --create-index

    ./elastic-index.pl --index-name kcls-1 --populate

    ./elastic-index.pl --index-name kcls-1 --activate-index

---

# KCLS Production Setup

* Two dedicated VMs with ~100G disk and 24G RAM
* Load-balanced with one write node, one replica node.
* A full index uses about 36G disk
* Apply firewall (iptables) to limit port 9200 access
* 2 Indexer Scripts

---

# Sysadmin Tools

    !sh

    curl -s http://localhost:9200/bib-search/_doc/891066 | jq -C . | less -R

    curl -s -XGET 'localhost:9200/bib-search/_search?pretty&q=dogs' | jq -C . | less -R

    curl -s -XGET 'localhost:9200/bib-search/_count?pretty' 

    curl -s -XGET 'localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason'

    curl -s -XGET 'localhost:9200/_cluster/health?pretty'

    #

---

# Testing Analysis

    !sh
    $ curl -s -XGET "localhost:9200/bib-search/_analyze?pretty" -H 'Content-Type: application/json' -d'
    {
      "analyzer" : "icu_folding",
      "text" : "En̲ iruḷ vān̲il oḷi nilavāy nī"
    }
    ' | jq -C . | less -R

    #

---

# Questions & Comments



