>A sample Rails application can be found on GitHub [https://github.com/searchly/searchly-rails-sample](https://github.com/searchly/searchly-rails-sample).

### Configuration

Ruby on Rails applications will need to add the following entry into their `Gemfile`.

```ruby
gem 'elasticsearch-model'
gem 'elasticsearch-rails'
```
Update application dependencies with bundler.
```sh
$ bundle install
```
Configure Rails Elasticsearch in `configure/application.rb` or `configure/environment/production.rb`

```ruby
Elasticsearch::Model.client = Elasticsearch::Client.new host: ENV['SEARCHBOX_URL']
```

### Index Creation

First add required mixin to your model;

```ruby
class Document < ActiveRecord::Base
  include Elasticsearch::Model
end
```

From Rails console, create `documents` index for model `Document`.

```ruby
Document.__elasticsearch__.create_index! force: true
```

### Search

Make your model searchable:

```ruby
class Document < ActiveRecord::Base
  include Elasticsearch::Model
  include Elasticsearch::Model::Callbacks
end
```

When you now save a record:

```ruby
Document.create name: "Cost",
                text: "Cost is claimed to be reduced and in a public cloud delivery model capital expenditure is converted."
````

The included callbacks automatically add the document to a `documents` index, making the record searchable:

```ruby
@documents = Document.search('Cost').records
```

Elasticsearch Ruby/Rails has very detailed documentation at official [Elasticsearch page](https://www.elastic.co/guide/en/elasticsearch/client/ruby-api/current/index.html).
