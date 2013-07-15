# Searchkick

Search made easy

## Usage

Searchkick provides sensible search defaults out of the box.  It handles:

- stemming - `tomatoes` matches `tomato`
- special characters - `jalapenos` matches `jalapeños`
- extra whitespace - `dishwasher` matches `dish washer`
- misspellings - `zuchini` matches `zucchini`

### Make Searches Better Over Time

Use analytics on search conversions to improve results.

Also, give popular documents a little boost.

### Zero Downtime Changes

Elasticsearch has a feature called aliases that allows you to change mappings with no downtime.

```ruby
Book.tire.reindex
```

This creates a new index `books_20130714181054` and points the `books` alias to the new index when complete - an atomic operation :)

**First time:** If books is an existing index, it will be replaced by an alias.

Searchkick uses `find_in_batches` to import documents.  To filter documents or eagar load associations, use the `tire_import` scope.

```ruby
class Book < ActiveRecord::Base
  scope :tire_import, where(active: true).includes(:author, :chapters)
end
```

There is also a rake task.

```sh
rake searchkick:reindex CLASS=Book
```

Thanks to Jaroslav Kalistsuk for the [original source](https://gist.github.com/jarosan/3124884).

Clinton Gormley also has a [good post](http://www.elasticsearch.org/blog/changing-mapping-with-zero-downtime/) on this.

## Elasticsearch Gotchas

### Mappings

When changing the mapping in a model, you must create a new index for the changes to take place.  Elasticsearch does not support updates to the mapping.  For zero downtime, use the `reindex` method above which creates a new index and swaps it in once built. To see the current mapping, use:

```sh
curl "http://localhost:9200/books/_mapping?pretty=1"
```

### Inconsistent Scores

Elasticsearch was built to be distributed.  However, with a low number of documents, the default settings can give us incorrect results.  You can [read more about it here](http://www.elasticsearch.org/blog/understanding-query-then-fetch-vs-dfs-query-then-fetch/).  To fix this, set the search type to `dfs_query_and_fetch`.  Alternatively, you can just use one shard with `settings: {number_of_shards: 1}`.

## Installation

Add this line to your application's Gemfile:

```ruby
gem "searchkick"
```

And then execute:

```sh
bundle
```

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request