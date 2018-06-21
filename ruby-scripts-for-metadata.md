# Library of Sample Scripts for SDR Metadata Management

## Splitting a single JSON file into many individual JSON records
```
## Use this script to take a single .JSON file that has many individual JSON records, split them into single files that are nested within the file-folder structure that NYU uses for OpenGeoMetadata.

require 'json'

irb_context.echo = false

nyu_file = File.read('/Users/andrewbattista/Downloads/UAE_complete_metadata_singlefile.json')

## The file where the original .json that contains all of the records is above. Make sure to include the full path

parsed_nyu = JSON.parse(nyu_file)

## The JSON.parse function parses the single file into discrete outputs as JSON files

parsed_nyu.each do |record|
  folder_name = record['layer_slug_s']
  dir1 = record['layer_slug_s'][4..7]
  dir2 = record['layer_slug_s'][9]
  dir3 = record['layer_slug_s'][10..11]
  dir4 = record['layer_slug_s'][12..13]

## This defines variables that correspond to the text string of the layer_slug_s element

  full_folder = "/Users/andrewbattista/Desktop/edu.nyu/handle"

  `mkdir -p #{full_folder}/#{dir1}/#{dir2}/#{dir3}/#{dir4}`

  ## This command above runs a unix command to make new folders according to the naming convention laid out from the named variables above

  File.open("#{full_folder}/#{dir1}/#{dir2}/#{dir3}/#{dir4}/geoblacklight.json", "w") do |f|

  ## We are opening a file that doesn't yet exist, giving it a name before it comes into being, and that name is the value full_folder/geoblacklight.json. Then it writes that to a file.

    f.write(JSON.pretty_generate(record))
  end

end
```
## Deleting a key from all metadata records in a repository

```bash
## This script can be used to delete a single key from a JSON file if it's not needed (e.g. nyu_addl_format_sm) for items that need to go back into OpenGeoMetadata

require 'json'

collection = JSON.parse(File.read("/Users/staff/Downloads/GlobalMap_singlefile.json"))


collection.each do |record|
  record.delete("nyu_addl_format_sm")
end

File.open("/Users/staff/Downloads/GlobalMap_singlefile.json", "w") do |f|
  f.write(JSON.pretty_generate(collection))
end
```

## Indexing metadata records into production instance of Solr

```ruby 
require 'rsolr'
require 'uri'
require 'find'
require 'json'

# irb_context.echo = false

PROD_MD_DIR = '/home/ubuntu/metadata/production'
PROD_SOLR_URL = 'http://54.174.220.44:8983/solr/blacklight_core'

def add_nyu_fields(record)
  if record['nyu_addl_format_sm'].nil?
    mod = record.dup
    mod['nyu_addl_format_sm'] = [ mod['dc_format_s'] ]
    return mod
  else
    return record
  end
end

def filter(record)
  return true if ((record['dct_provenance_s'] == 'NYU') || (record['dc_rights_s'] == 'Public'))
  return false
end


## Find all `geoblacklight.json` records
gbl = Find.find(PROD_MD_DIR).select{ |x| File.basename(x) == 'geoblacklight.json' || File.basename(x) == 'collection.json'}

puts "Found #{gbl.count} record files in: #{PROD_MD_DIR}"

filtered_records = [] ## A place to store them

gbl.each do |path|
  if File.basename(path) == "geoblacklight.json"
    record = JSON.parse(File.read(path)) ## Read and parse the record
    if (filter(record)) ## See if we want it; you can change the variables depending on how you want to filter
      filtered_records << add_nyu_fields(record)
    end
  elsif File.basename(path) == "collection.json"
    records = JSON.parse(File.read(path)) ## Read and parse the record
    filtered_records.concat records.select{ |x| filter(x) }.map{ |x| add_nyu_fields(x) }
  end
end

puts "Found #{filtered_records.count} usable records."

## Create a connection object
solr = RSolr.connect :url => PROD_SOLR_URL

## this Solr URL points to the local instance of Solr

## Optionally, confirm that you are connected
#results = solr.get 'select', :params => {:q => '*:*'}

completed_counter = 0

filtered_records.each_slice(100) do |slice|
  begin
    retries ||= 0
    solr.add slice
    solr.commit
    completed_counter += slice.length
    puts "[ #{completed_counter.to_f / filtered_records.count} ] #{completed_counter}/#{filtered_records.count} records have been indexed..."
  rescue
    puts "#### Failure! Check logs! ####"
    retry if (retries += 1) < 2
  end
end
```
