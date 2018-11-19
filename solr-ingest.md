# Sample Scripts for Solr Ingest

### Table of Contents

* [Filtering records / indexing into Solr](#Filtering records / indexing into Solr)

#### Filtering records / indexing into Solr

```ruby
require 'rsolr'
require 'uri'
require 'find'
require 'json'

PROD_MD_DIR = '/home/ubuntu/metadata/production'

## Find all `geoblacklight.json` records
gbl = Find.find(PROD_MD_DIR).select{ |x| File.basename(x) == 'geoblacklight.json'}

puts "Found #{gbl.count} records in: #{PROD_MD_DIR}"

filtered_records = [] ## A place to store them

gbl.each do |path|
  record = JSON.parse(File.read(path)) ## Read and parse the record
  if (record['dct_provenance_s'] == 'NYU') || (record['dc_rights_s'] == 'Public') ## See if we want it
    filtered_records << record
  end
end
```

At this point, we should have all of the records that we're interested in sending to Solr stored within `filtered_records`.

It's never a bad idea to take a closer look, and make sure `filtered_records` contains what you think it does. Try inspecting elements at random. Better yet, slice through the list and check some properties:
```ruby
filtered_records.each_slice(100) do |slice|
  ## Now I'll check the first element of each slice:
  puts "#{slice[0]['dc_rights_s']} -- #{slice[0]['dc_identifier_s']}"
end
```

Looks good? Ok. Now you can use [`RSolr`](https://github.com/rsolr/rsolr).

**Note** that the following uses the actual production Solr core URL. This will only be accessible from `P1_1:metadata`, or if you opened a port in the firewall of `P1_1:solr-core`.

```ruby
## Create a connection object
solr = RSolr.connect :url => "http://54.174.220.44:8983/solr/blacklight_core"

## Optionally, confirm that you are connected
results = solr.get 'select', :params => {:q => '*:*'}
puts results
```

To actually index these records, you can use the following code. But **be careful**, particularly when you are interacting with the production Solr core! **Always try indexing to a development core first.**

```ruby
filtered_records.each_slice(100) do |slice|
  solr.add slice
  solr.commit
end
```
