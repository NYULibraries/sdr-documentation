# Scripts for Metadata Management

This section is a gathering of scripts for commonly used metadata editing and transformation processes.

### Table of Contents
- [Splitting a single JSON file into many individual JSON records](#Splitting-a-single-JSON-file-into-many-individual-JSON-records)
- [Deleting a Key from all Metadata Records in a Repository](#Deleting-a-Key-from-all-Metadata-Records-in-a-Repository)
- [Appending one or more Subjects or other elements onto an existing batch of GeoBlacklight records](#Appending-one-or-more-Subjects-or-other-elements-onto-an-existing-batch-of-GeoBlacklight-records)
- [Adding a key-value URL in the references field to one or more records](#Adding-a-key-value-URL-in-the-references-field-to-one-or-more-records)


#### Splitting a single JSON file into many individual JSON records
 
```ruby
## Use this script to take a single .JSON file that has many individual JSON records, split them into single files that are nested within the file-folder structure that NYU uses for OpenGeoMetadata.

require 'json'

irb_context.echo = false

nyu_file = File.read('/Users/staff/Downloads/metadata_singlefile.json')

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

  full_folder = "/Users/staff/Desktop/edu.nyu/handle"

  `mkdir -p #{full_folder}/#{dir1}/#{dir2}/#{dir3}/#{dir4}`

  ## This command above runs a unix command to make new folders according to the naming convention laid out from the named variables above

  File.open("#{full_folder}/#{dir1}/#{dir2}/#{dir3}/#{dir4}/geoblacklight.json", "w") do |f|

  ## We are opening a file that doesn't yet exist, giving it a name before it comes into being, and that name is the value full_folder/geoblacklight.json. Then it writes that to a file.

    f.write(JSON.pretty_generate(record))
  end

end
```
#### Deleting a Key from all Metadata Records in a Repository

This script can be used to delete a single key from a JSON file if it's not needed (e.g. nyu_addl_format_sm) for items that need to go back into OpenGeoMetadata, or any other key that persists after harvesting metadata from another source.

```ruby
require 'json'

collection = JSON.parse(File.read("/Users/staff/Downloads/GlobalMap_singlefile.json"))

collection.each do |record|
  record.delete("nyu_addl_format_sm")
  
  # Here, obviously change the key accordingly to delete what you want
end

File.open("/Users/staff/Downloads/GlobalMap_singlefile.json", "w") do |f|
  f.write(JSON.pretty_generate(collection))
end
```

#### Appending one or more Subjects or other elements onto an existing batch of GeoBlacklight records

```ruby
## this script allows you to add one or more subjects to batches of existing GeoBlacklight records that meet a certain criteria.

require 'find'
require 'json'
conf.echo = false ## This just disables print-outs of every return value

PATH_TO_EDU_NYU = "/Users/staff/desktop/edu.nyu"

record_paths = Find.find(PATH_TO_EDU_NYU).select{ |x| x.include? "geoblacklight.json" }

record_paths.each_with_index do |path, idx|
  record = JSON.parse(File.read(path))
  ## At this point, all of the code below will be executed on every
  ## single record path.

  ## The variable `record` contains the parsed GeoBlacklight record.

  if record['dct_isPartOf_sm'].include? "Soviet Military Topographic Maps" ## obviously change the condition here depending on which batch you want to alter
    record['dc_subject_sm'].concat ["FIST_SUBJECT", "SECOND_SUBJECT"] ## obviously change the subjects inside to what you want.

    ## We also now need to save the modified record:
    File.open(path, "w") do |f|
      f.write(JSON.pretty_generate(record))
    end
  end

end
```

#### Adding a key-value URL in the references field to one or more records

```ruby
## This script allows you to add a references URL key-value to an existing GeoBlacklight record or records based on a conditon.

require 'find'
require 'json'
conf.echo = false ## This just disables print-outs of every return value

PATH_TO_EDU_NYU = "/Users/staff/desktop/edu.nyu" ## obviously, change this path to match the location of the repository on your own computer

record_paths = Find.find(PATH_TO_EDU_NYU).select{ |x| x.include? "geoblacklight.json" }

record_paths.each_with_index do |path, idx|
  record = JSON.parse(File.read(path))
  ## At this point, all of the code below will be executed on every
  ## single record path.

  ## The variable `record` contains the parsed GeoBlacklight record.

  if record['dct_isPartOf_sm'].include? "Soviet Military Topographic Maps"
    ## First we need to parse the JSON object from the references string
    references = JSON.parse(record['dct_references_s'])

    ## Now we add (or replace) the codebook entry
    references["http://lccn.loc.gov/sh85035852"] = "https://archive.nyu.edu/bitstream/2451/37402/2/nyu_2451_37402_doc.zip"

    ## Finally, we "reverse" the first step, and save a JSON escaped version
    ## of the references object into a string
    record['dct_references_s'] = JSON.generate(references)

    ## We also now need to save the modified record:
    File.open(path, "w") do |f|
      f.write(JSON.pretty_generate(record))
    end
  end

end
```
