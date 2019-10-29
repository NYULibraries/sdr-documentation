# Code Snippets for File / Folder Management

### Table of Contents

* [Unzipping nested .zip files, then deleting the .zip](#Unzipping-nested-.zip-files,-then-deleting-the-.zip)
* [Creating empty folder containers for data downloads](#Creating-empty-folder-containers-for-data-downloads)
* [Delete all files of a kind within a directory](#Delete-all-files-of-a-kind-within-a-directory)
* [Finding all files within a directory](#Finding-all-files-within-a-directory)
* [Zipping all files in a directory according to the folder that contains them](#Zipping-all-files-in-a-directory-according-to-the-folder-that-contains-them)
* [Turn off Ruby interpreter echo](#Turn-off-Ruby-interpreter-echo)
* [Find and replace on huge text files](#Find-and-replace-on-huge-text-files)

### Unzipping nested .zip files, then deleting the .zip

Occassionally, you will need to expose all parts of files in an archive. Use this script to do that and also delete the archives that contain the files.

```ruby
require 'find'

FOLDER_WITH_CONTAINERS="/Users/staff/Desktop/sdr_collection"

zip_paths = Find.find(FOLDER_WITH_CONTAINERS).select{ |path| File.extname(path) == ".zip"}

## For each zip path, we determine its parent directory, and then
## run bash commands to unzip and delete the zip archive

zip_paths.each do |zp|
  containing_directory = zp.gsub(File.basename(zp),"")
  `cd #{containing_directory}; unzip #{zp}; rm #{zp}`
end
```

### Creating empty folder containers for data downloads

When you want to harvest data from an open data repository or some other structured source, you need an efficient way to download files as they are originally named and store them. This script allows you to create empty download containers that are named according to the unique identifier of your download source. It assumes that you have a `singlefile.json` of many metadata records that are named according to unique ID.

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
Note that this script actually writes a file called `geoblacklight.json` with the contents of each record and places it within a containing directory. You may want to delete these. If so, navigate to the directory where all the folders are and run the following:

```bash
find . -name 'geoblacklight.json' type -f -delete
```
### Delete all files of a kind within a directory

You may want to get rid of all files in a directory, or all of one particular kind of file within a directory. If so, navigate to the containing directory and run the following:

```bash
find . -name '*.json' type -f -delete
## Deletes all files that end with the extension .json within the directory
```

### Finding all files within a directory

`find` does a lot, but most importantly, it allows us to search through a directory to list all of the files that exist in it or any of its subdirectories. This is an important first step in assessing a collection and its contents.

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

### Zipping all files in a directory according to the folder that contains them

A common task is to zip all files within a directory into an archive, name the archive according to the folder that contains them, and to do this recursively. In order to do this, navigate to the directory in question and run the following:

```bash
for i in */; do zip -r "${i%/}.zip" "$i"; done
```
The result leaves the original files in place, but adds a directory with the files compressed.

#### Turn off Ruby interpreter echo

When writing Ruby scripts, use the following to disable print outs to the console. This can be useful when manipulating huge files.
```ruby
irb_context.echo = false
```

#### Find and replace on huge text files

When scraping or dealing with massive .JSON files (for example), a regular text editor like Atom is going to crash if you try to load it or use it to find and replace. Better to use vi:

```
:%s/search_string/replacement_string/g
```
