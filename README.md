# Introduction

The embedded Milvus broughts up a Milvus instance on that starts and exits whenever you wish it to, while keeping all data and logs persistent. You can have it work within two simple steps:

```python
$ pip install milvus
$ (in python) >>> import milvus
```

Embedded milvus does not have any other dependencies and do not require anything pre-installed, including Etcd, MinIO, etc.

Everything you do with embedded Milvus, every piece of code you write for embedded Milvus can be safely migrated to other forms of Milvus (standalone version, cluster version, cloud version, etc.) or simply "Write once, run anywhere".

Please note that it is not suggested to use embedded Milvus in a production environment. Consider using Milvus clustered or the fully managed Milvus on Cloud. 

## Configuration

A configurable file will be created on initial start located at `/tmp/milvus/configs/embedded-milvus.yaml`

## Data and Log Persistency

All data and logs are persistent and will be stored under `/tmp/milvus/` by default. If you want them somewhere else, you can update the embedded Milvus configuration file.

## Working with PyMilvus

Embedded Milvus depends on PyMilvus. We are keeping our PyMilvus version consistent with the embedded Milvus version.

## Release Plan

Embedded Milvus is released together with the main Milvus project and will adopt Milvus's version as its own version.

Embedded Milvus always depends on the most suitable PyMilvus version when released.

# Requirements

```shell
python >= 3.9
```

# Installation

You can install embedded Milvus via `pip` or `pip3` for Python 3.6+:

```shell
$ pip3 install milvus
```

You can install a specific version of embedded Milvus by:

```shell
$ pip3 install milvus==2.0.1
```

You can upgrade embedded Milvus by:

```shell
$ pip3 install --upgrade milvus
```

# Running Embedded Milvus

You can start by importing milvus:

```python
$ python3
Python 3.9.10 (main, Jan 15 2022, 11:40:53)
[Clang 13.0.0 (clang-1300.0.29.3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import milvus
>>>
```

Milvus is now ready and you can start interacting with it. For a full example, you can look at [Hello Milvus](https://milvus.io/docs/v2.0.0/example_code.md).

```python
$ python3
Python 3.9.10 (main, Jan 15 2022, 11:40:53)
[Clang 13.0.0 (clang-1300.0.29.3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import milvus # embedded Milvus ready to go
>>>
>>> import random
>>> from pymilvus import (
...     connections,
...     utility,
...     FieldSchema, CollectionSchema, DataType,
...     Collection,
... )
>>> connections.connect("default", host="localhost", port="19530")
>>> has = utility.has_collection("hello_milvus")
>>> fields = [
...     FieldSchema(name="pk", dtype=DataType.INT64, is_primary=True, auto_id=False),
...     FieldSchema(name="embeddings", dtype=DataType.FLOAT_VECTOR, dim=8)
... ]
>>> schema = CollectionSchema(fields, "hello_milvus is the simplest demo to introduce the APIs")
>>> hello_milvus = Collection("hello_milvus", schema, consistency_level="Strong")
>>> num_entities = 3000
>>> entities = [
...     [i for i in range(num_entities)], # provide the pk field because `auto_id` is set to False
...     [[random.random() for _ in range(8)] for _ in range(num_entities)],  # field embeddings
... ]
>>> insert_result = hello_milvus.insert(entities)
>>> index = {
...     "index_type": "IVF_FLAT",
...     "metric_type": "L2",
...     "params": {"nlist": 128},
... }
>>> hello_milvus.create_index("embeddings", index)
>>> hello_milvus.load()
>>> vectors_to_search = entities[-1][-2:]
>>> search_params = {
...     "metric_type": "l2",
...     "params": {"nprobe": 10},
... }
>>> result = hello_milvus.search(vectors_to_search, "embeddings", search_params, limit=3)
>>> for hits in result:
...     for hit in hits:
...         print(f"hit: {hit}")
...
hit: (distance: 0.0, id: 2998)
hit: (distance: 0.1088758111000061, id: 2345)
hit: (distance: 0.12012234330177307, id: 1172)
hit: (distance: 0.0, id: 2999)
hit: (distance: 0.0297045037150383, id: 2000)
hit: (distance: 0.16927233338356018, id: 560)
>>> utility.drop_collection("hello_milvus")
```

You can also start another python script or SDK to work with embedded Milvus while it is not closed. For example, in a new shell window, you could:

```shell
$ python3 ./examples/hello_milvus
```

Finally, when you are done, simply do exit().

```python
>>> exit()
$ 
```

# Building the Package

1. In Milvus repository, build it with:
```shell
$ make embedded-milvus
```

Upon success, a dynamic library (embd-milvus.so and embd-milvus.h) will be created, create a new folder `bin` and put these two files under it. 

```shell
embd-milvus/
├── LICENSE
├── README.md
├── examples
│   └── hello_milvus.py
├── milvus
│   ├── __init__.py
│   ├── bin
│   │   ├── embd-milvus.h
│   │   └── embd-milvus.so
│   ├── configs
│   │   └── embedded-milvus.yaml
│   └── milvus.py
└── setup.py
```

2. Build the wheel:

```shell
$ python3 setup.py bdist_wheel
```

3. Test it locally

Under the embd-milvus directory, start a virtual environment:

```shell
$ python3 -m pip install virtualenv
$ virtualenv venv
$ source venv/bin/activate
# Force install the wheel you just built in the last step.
(venv) $ pip install --upgrade --force-reinstall ./dist/milvus-2.0.1-{python}-{abi}-{platform}.whl
(venv) $ pyton3
Python 3.9.10 (main, Jan 15 2022, 11:40:53)
[Clang 13.0.0 (clang-1300.0.29.3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import milvus
...
```

