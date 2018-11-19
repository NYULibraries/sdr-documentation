# Code Snippets for File / Folder Management

### Table of Contents

* [Unzipping nested .zip files, then deleting the .zip](#Unzipping nested .zip files, then deleting the .zip)
* [Creating empty folder containers for data downloads](#Creating empty folder containers for data downloads)
* [Finding all files within a file-folder directory](#Finding all files within a file-folder directory)


### Unzipping nested .zip files, then deleting the .zip

Occassionally, you will need to expose all parts of files in an archive (usually to determine how many shapefiles are in a given collection). Use this script to do that and also delete the archives that contain the files.

```ruby
require 'find'

FOLDER_WITH_CONTAINERS="/Users/staff/Desktop/sdr_collection"

## Gather all paths, recursively, from the folder above, but only
## retain those ending with `.zip`

zip_paths = Find.find(FOLDER_WITH_CONTAINERS).select{ |path| File.extname(path) == ".zip"}

## For each zip path, we determine its parent directory, and then
## run bash commands to unzip and delete

zip_paths.each do |zp|
  containing_directory = zp.gsub(File.basename(zp),"")
  `cd #{containing_directory}; unzip #{zp}; rm #{zp}`
end
```

### Creating empty folder containers for data downloads

When you want to harvest data from an open data repository or some other structured source, you need an efficient way to download files as they are originally named and store them. This script allows you to create empty download containers that are named according to the unique identifier of your download source. It assumes that you have a singlefile.json of many metadata records that are named according to unique ID.

```ruby
require 'json'

irb_context.echo = false

nyu_file = File.read(‘/Users/staff/Downloads/nola.json’)

## The file where the original singlefile.json that contains all of the records is above. Change path accordingly.

parsed_nyu = JSON.parse(nyu_file)

## The JSON.parse function parses the singlefile.json into discrete outputs as JSON files

parsed_nyu.each do |record|
 folder_name = record[‘id’]
 dir1 = record[‘id’]

## The above declares that each folder will be named according to the ['id'] element in each respective .JSON file. It's up to you to determine what that element is called in your file and change accordingly. Change it in both variables.

 full_folder = “/Users/staff/Desktop/nola-open-data”

 `mkdir -p #{full_folder}/#{dir1}`

 ## The above runs a unix command to make new folders according to the naming convention laid out from the named variables above and stores them in the place you stipulate on your computer.
 
 File.open(“#{full_folder}/#{dir1}/geoblacklight.json”, “w”) do |f|

   ## We are opening a file that doesn’t yet exist, giving it a name before it comes into being, and that name is the value full_folder/geoblacklight.json. Then it writes that to a file. This script places geoblacklight.json files into each containing folder. If you want to delete them, use the delete script below.

     f.write(JSON.pretty_generate(record))

 end

end
```

### Delete all kinds of files within a directory recursively

### Finding all files within a file-folder directory

`find` does a lot, but most important for us is that it allows us to recursively search through a directory to list all of the files that exist in it or any of its subdirectories. This is an important first step in assessing a collection and its contents.

```bash
## Find all files on your desktop or whichever directory you want

cd ~/Desktop

find .

## the . just means current directory

```

Note that you can write the output of the above to a text file, instead of to the console:

```bash
find . > ~/Desktop/find_result.txt

## Creates a new file containing the results
```

You can also add name restrictions to `find`; for instance, maybe you want to gather a list of all files within a directory (_and_ any of its subdirectories) which are ESRI Shapefiles. Therefore, we are only interested in files ending in `.shp`:

```bash
cd east_view_collection

# this assumes that there's a folder called east_view_collection that has a bunch of shapefiles in it

find . -name "*.shp"

# this recursively locates all files matching the given pattern

find . -name "*.shp" > ~/Desktop/east_view_files.txt

# this saves the result to a text file, which makes it easy to see how many shapefiles exist
```

### Filtering records / indexing into Solr

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


### Addendum: General Ruby Tricks

#### Turn off Ruby interpreter "echo"

```ruby
irb_context.echo = false
```

#### Consume a list in batches
```ruby
some_list = (1..10000).to_a ## makes a list from the range 1 -> 10000
some_list.each_slice(100) do |slice| ## iterates over the list in slices of size 100
  puts slice[0] ## print the first element of each slice/batch
end
```
