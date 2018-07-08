+++
draft = true
tags = ["elasticsearch","dataengineering"]
categories = ["data"]
description = ""
date = "2018-04-08T01:03:50+09:00"
title = "ElasticSearch"
author = "jahyun.koo"
+++

ElasticSearch는 Lucene 기반 오픈소스 검색엔진이다. 사용하기 쉬운 API와 빠른 성능을 갖고 있어 가장 많이 쓰는 검색엔진이다.

# Elasticsearch 

### What Elasticsearch is not

Elasticsearch is not a primary data store. Although it’s technically possible, there’s no guarantee that your data will be correct. Each document has a version number that increases monotonically. When two calls write to Elasticsearch, both will get written simultaneously, but only one will be the latest version. Out of the box, Elasticsearch does not support ACID transactions.

# 검색엔진

# 사용처

# 

알고리즘과 자료구조를 이해하는건 큰 도움이 된다.

- How simple searches are performed
- What types of searches can (and cannot) effectively be done, and why, with an inverted index, we transfrom problems until they look like string-prefix problems
- Why text processing is important
- How indexes are built in "segments" and how that affects searching and updating
- What constitutes a Lucene-index
- The Elasticsearch shard and index

# 내부 구조

# Inverted Indexes and Index Terms

# Doc Values

# Fielddata
