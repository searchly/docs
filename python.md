# Using Searchly with Python

Elasticsearch provides two types of clients for Python [low-level](https://elasticsearch-py.readthedocs.io/en/master/)
and [elasticsearch-dsl](https://elasticsearch-dsl.readthedocs.io/en/latest/).


## Low-level client

### Compatibility

The library is compatible with all Elasticsearch versions since `0.90.x` but you have to use a matching major version:

Set your requirements in your setup.py or requirements.txt is:

```sh
# Elasticsearch 6.x
elasticsearch>=6.0.0,<7.0.0

# Elasticsearch 5.x
elasticsearch>=5.0.0,<6.0.0

# Elasticsearch 2.x
elasticsearch>=2.0.0,<3.0.0
```

Install via pip;

```sh
$ pip install elasticsearch
```

### Connection & Authentication

```python

from elasticsearch import Elasticsearch

es = Elasticsearch(['https://site:api-key@xyz.searchly.com:443'])
```

### Example Usage

```python

from datetime import datetime
from elasticsearch import Elasticsearch
es = Elasticsearch(['https://site:api-key@xyz.searchly.com:443'])

doc = {
    'author': 'kimchy',
    'text': 'Elasticsearch: cool. bonsai cool.'
}

# ignore 400 cause by IndexAlreadyExistsException when creating an index
es.indices.create(index='test-index', ignore=400)

# create tweet
res = es.index(index="test-index", doc_type='tweet', id=1, body=doc)
print(res['created'])

# get tweet
res = es.get(index="test-index", doc_type='tweet', id=1)
print(res['_source'])

# search tweet
res = es.search(index="test-index", body={"query": {"match_all": {}}})
print("Got %d Hits:" % res['hits']['total'])
for hit in res['hits']['hits']:
    print("%(author)s: %(text)s" % hit["_source"])

```

## DSL (high-level client)

The library is compatible with all Elasticsearch versions since `1.x` but you have to use a matching major version:

```python
# Elasticsearch 6.x
elasticsearch-dsl>=6.0.0,<7.0.0

# Elasticsearch 5.x
elasticsearch-dsl>=5.0.0,<6.0.0

# Elasticsearch 2.x
elasticsearch-dsl>=2.0.0,<3.0.0

# Elasticsearch 1.x
elasticsearch-dsl<2.0.0
```

### Connection & Authentication

You can either manually pass your connection instance to each request or set default connection for all requests.

#### Manual

```python
s = Search(using=Elasticsearch('https://site:api-key@xyz.searchly.com:443'))
```

#### Default connection
To define a default connection that will be used globally, use the connections module and the create_connection method:

```python
from elasticsearch_dsl import connections

connections.create_connection(hosts=['https://site:api-key@xyz.searchly.com:443'], timeout=20)
```


### Indexing

You can create model-like wrapper for your documents via using `DocType` class.
It can provide mapping and settings for Elasticsearch.

```Python
from elasticsearch_dsl import DocType, Date, Nested, Boolean, \
    analyzer, InnerDoc, Completion, Keyword, Text

html_strip = analyzer('html_strip',
    tokenizer="standard",
    filter=["standard", "lowercase", "stop", "snowball"],
)

class Comment(InnerDoc):
    author = Text(fields={'raw': Keyword()})
    content = Text(analyzer='snowball')

class Post(DocType):
    title = Text()
    title_suggest = Completion()
    published = Boolean()
    category = Text(
        analyzer=html_strip,
        fields={'raw': Keyword()}
    )

    comments = Nested(Comment)

    class Meta:
        index = 'blog'

    def add_comment(self, author, content):
        self.comments.append(
          Comment(author=author, content=content))

    def save(self, ** kwargs):
        return super().save(** kwargs)

```

Also you can explicitly create index and provide settings at index;

```Python

from elasticsearch_dsl import Index, DocType, Text, analyzer

blogs = Index('blogs')

# define aliases
blogs.aliases(
    old_blogs={}
)

# register a doc_type with the index
blogs.doc_type(Post)

# can also be used as class decorator when defining the DocType
@blogs.doc_type
class Post(DocType):
    title = Text()

# You can attach custom analyzers to the index

html_strip = analyzer('html_strip',
    tokenizer="standard",
    filter=["standard", "lowercase", "stop", "snowball"],
)

blogs.analyzer(html_strip)

# delete the index, ignore if it doesn't exist
blogs.delete(ignore=404)

# create the index in elasticsearch
blogs.create()

```

Create and save documents to Elasticsearch;

```Python
# instantiate the document
first = Post(title='My First Blog Post, yay!', published=True)
# assign some field values, can be values or lists of values
first.category = ['everything', 'nothing']
# every document has an id in meta
first.meta.id = 47


# save the document into the cluster
first.save()
```

### Searching

``` python
# by calling .search we get back a standard Search object
s = Post.search()
# the search is already limited to the index and doc_type of our document
s = s.filter('term', published=True).query('match', title='first')


results = s.execute()

# when you execute the search the results are wrapped in your document class (Post)
for post in results:
    print(post.meta.score, post.title)
```

