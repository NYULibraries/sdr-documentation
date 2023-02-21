---
layout: post
title:  "Post 5 - SDR Collection Workflow"
date:   2018-06-12 10:48:45 -0500
author: Andrew Battista and Stephen Balogh
categories: geoblacklight
---

# SDR Collection / Acquisitions Workflow
Stephen Balogh <sgb334@nyu.edu> & Andrew Battista <ab6137@nyu.edu>

### Note

Many of the examples in this document make use of a Ruby gem called **SdrFriend**, so named because of how friendly it is. The codebase for that tool, as well as documentation (including on how to set it up), can be found at its [Github page](https://github.com/sgbalogh/sdrfriend). In order to install this gem, make sure you have:

- The latest version of Ruby installed using [Ruby Version Manager](https://rvm.io/rvm/install)
- HomeBrew [installed](https://brew.sh/)
- Installed the bundler gem by entering `gem install bundler`

 E mail Andrew Battista to get a copy of **SdrFriend** Then, download the repository, navigate to it and enter `bundle install`

## 1. Assessment

The collection and acquisition process begins by assessing the scope of a batch of data that you seek to acquire. The easiest way to do this is to start a spreadsheet and track out the number of layers that exist within a given collection.

#### Example Scenario
We acquire a hard disk of Shapefile (vector) layers from EastView. There are some 300 layers, grouped arbitrarily into a directory structure. First we want to get an idea of how many discrete layers (or individual Shapefiles) there are. Maybe this information is already contained in a spreadsheet provided by the vendor; if not, we may have to generate a collection of all shapefiles.

```bash
cd east_view_collection
# this assumes that there's a folder called east_view_collection that has a bunch of shapefiles in it
find . -name "*.shp"
# this recursively locates all files matching the given pattern
find . -name "*.shp" > ~/Desktop/east_view_files.txt
# this saves the result to a text file, which makes it easy to see how many shapefiles exist
```
The above process is just one way to determine how many layers exist in a collection. You could also count these manually, or view the list of files in Finder, select them all, then hit ``command+c`` and then paste the result into a spreadsheet. Obviously, the kind of file you're collecting matters, so you should adjust accordingly.

## 2a. Creating Repository Records / "Minting Handles"

Once the number of items in a given collection is determined, the next step is to "mint" the requisite number of IDs from the FDA. Note that all collection additions should follow this minting process from the start. **Do not create an item in the SDR using the DSpace interface.** Instead, even if you are only creating one item, use **SdrFriend** and update the Handle table immediately.

SDR assets span two different collections on the FDA, both located within the [Division of Libraries]("https://archive.nyu.edu/handle/2451/14821") community. They are:
 - [Spatial Data Repository]("https://archive.nyu.edu/handle/2451/33902") `dspace id: 651`
  -- By default, all items in this collection are public and world-accessible; accordingly, only open-data or non-licensed datasets should go here
 - [Spatial Data Repository (Private)]("https://archive.nyu.edu/handle/2451/33903") `dspace id: 652`
  -- By default, all items in this collection are visible only to logged in FDA users. The FDA is configured to use NYU Single Sign On (SSO), so in effect this means that all items in this collection are visible only to users with NYU login credentials

You should determine which of the two collections you want to create records in. Typically, in the course of accessioning a single collection, you will interact only with one of the two at a time.

**SdrFriend** can be used to mint FDA records in the relevant collection. In particular, the `fda:mint` rake task allows you to pre-allocate an arbitrary amount of records and save a receipt to a text file:

```bash
cd ~/git/SdrFriend
# The command bundle exec rake -T # allows you to view all available tasks in SdrFriend
bundle exec rake fda:mint[private,100,"/Users/sgb334/Desktop/new_handles.csv"]
# First part: Is the collection public or private? (in this case private)
# Second part: How many IDs do you want? (in this case 100)
# Third part: The path to where you want the new file to be saved and your name for it (in this case new_handles.csv)
```
If the above command is successful and no errors were thrown, you should have saved a CSV text file in the format  "*handle*,*dspace-internal-id*". You will need this file to update the `edu.nyu` lookup table, so make sure you save it. Note that you can also copy the output of the file as it appears in terminal, which you will paste in the `handle-dspace-lookup.csv` file that you will soon pull from GitHub (if you haven't already).

*Additional note: for each of these commands, spacing and capitalization matters, so it's easiest to copy them directly from this documentation. Another good tip is to drag the file needed to run a command into Terminal directly, as it will print out the correct file-folder path*

## 2b. Updating the Handle lookup table

The FDA (confusingly) uses two types of IDs to refer to records: an internal id (referred to as "dspace_id" in this document) and a Handle identifier. If only these were the same!

The Handles are great for public consumption, but unfortunately the REST API for the Faculty Digital Archive (i.e., the way in which programmed tools interact with the repository) requires the internal dspace_id, and there's no simple way to turn one into another. As a work around, we manage our own lookup table, stored in the [edu.nyu repository]("https://github.com/OpenGeoMetadata/edu.nyu/blob/master/misc/handle-dspace-lookup.csv") on GitHub.

**Make sure to update this table whenever you pre-allocate / "mint" new FDA records. Disassociating the dspace_id from the Handle identifier causes problems down the line**

One way to make updates to the lookup table is to set up git to commit to the `edu.nyu` metadata repository (see [these instructions on how to do it](https://help.github.com/articles/set-up-git/)). If you're already set up to commit to the edu.nyu repository, here's a simple way to update the table:

```bash
cd ~/git/edu.nyu
git checkout master
## Then, run this next command:
git pull
## this makes sure you have the most current version of the entire edu.nyu repository on your computer.
## Once you've done this, copy the direct output (file) of the handles and DSpace IDs as it appears in Terminal, or copy the text of the .CSV that was generated from the fda:mint command above
## Then, go to the lookup table folder within the misc folder in the edu.nyu repository on your drive, open the .CSV with Atom, and paste the new handles in there. Make sure there are no extra lines at the end, and save it.
## Inspect that everything worked by executing the following commands:

git status

git diff misc/handle-dspace-lookup.csv

## If it looks good, go ahead and commit it
git add ./misc/handle-dspace-lookup.csv ## stage the changes in the file
git commit -m "adds some new Handles for X collection"
git push
```
*Note: if any of the above looks confusing to you, there are a million other ways to update the table; you could try using the GitHub Desktop app, for instance. You could manually concatenate the two CSV documents using Excel (or even just a text editor like Atom). Just make sure that the lookup table is completely in sync any time new handles are minted, or else SdrFriend won't work anymore. Also note that at this point you may need to delete any .DS_Store files that may have crept in*

## 3. Create GeoBlacklight OpenGeoMetadata

At this point, you've got a grasp on what documents exist and need to be described, and you've already pre-allocated the appropriate number of FDA records in the relevant collection. Now you'll want to start actually constructing what will end up being the various GeoBlacklight records describing your layers.

These days, it seems like simply working in a spreadsheet is the best way to start. Begin by making a copy of the [GeoBlacklight template](https://docs.google.com/spreadsheets/d/1XAjL59fEsegZm2dArVjeT0XaOLohvDIkSQ9_RftSPlI/edit?usp=sharing). Alternatively, you can have **SdrFriend** generate one for you:

```bash
cd ~/git/SdrFriend
bundle exec rake metadata:template[/Users/sgb334/Desktop/new_collection.csv]
## this stipulates the path where you want to save the file and the name of the file
```
The first data you should add is the dspace_id and Handle attributes from the collection you just pre-allocated. Go ahead and paste them into the CSV. Feel free to add in additional columns to the CSV document if it's convenient to do so (such as "original file names"), though **only the columns listed in the template** will persist after converting the CSV to actual GeoBlacklight records. Also, very important: **don't rename any of the existing template columns.**

From here, do whatever you need to do to fill in the rest of the columns. In general, you are responsible for all of the metadata elements; the only ones which cannot be filled in at this time are the `ref:download-url` and `ref:documentation-url` fields, which are intended to point to FDA bitstreams. That workflow is described below. If you are using the Google Sheets template, the `layer_modified_dt` field should also automatically update as you work in the Sheet. If you have any questions about the rules or best practices for filling out metadata fields, refer to the [GeoBlacklight Schema](https://github.com/geoblacklight/geoblacklight/blob/master/schema/geoblacklight-schema.md). One of the most difficult fields to come up with is the `solr_geom`, so refer to the appendix of this document for an **SdrFriend** command to find the `solr_geom` of a given file if you aren't going to use another method.

Every FDA/GeoBlacklight layer needs at least one download url (the main, `ref:download-url` value). If your layer comes with additional documentation or codebook material, you may also want to link directly to that in the GeoBlacklight record (via `ref:documentation-download`), and make use of GeoBlacklight's contextual rendering for it. Unfortunately, though, it isn't possible to predict what the bitstream URLs will be until you have already uploaded them to your empty FDA container records. Thus, we will need to move on once we have enough metadata to otherwise constitute minimal viable GeoBlacklight records. Later, we will insert the correct download URLs before outputting the "OpenGeoMetadata-ready" version of the records.

## 4. Create bitstreams packages and prepare them

At this point, we now need prepare to upload our actual content to the digital repository (FDA). First, we need to assemble the bitstreams that we will be uploading. Although it isn't perfect, the best strategy is to create a "container" for our impending upload. To do this, run the following task in **SDRFriend**

```bash
bundle exec rake files:download_containers[/Users/andrewbattista/Desktop/containers_for_collection,Users/andrewbattista/Downloads/UAE_collection_metadata.csv]

## In the above command, you are first stipulating the area of your computer where you want to create the new folders and then the second part is the name of the CSV that you're using (from step 3 above) to make the GeoBlacklight metadata. Likely, you will have downloaded this from your Google Sheet. It is important that your CSV does not have spaces or numbers in its title

## Note also that this command just creates a structure of blank folders that are named according to the handles you've already created. After running this command, you'll need to do a similar thing to create documentation folders:

bundle exec rake files:documentation_containers[/Users/andrewbattista/Desktop/doc_containers_for_collection,Users/andrewbattista/Downloads/UAE_collection_metadata.csv]

```

After creating both sets of new folders, you may want to combine them into a single folder, or you may want to keep them separate. This is an assessment you'll have to make depending on the condition of the data you're taking in. Which leads us into the next point.

All SDR submissions have to be in a directory structure that looks like this:

```
- nyu_2451_12345/
    - DISTRICT91.shp
    - DISTRICT91.prj
    - DISTRICT91.shx
    - DISTRICT91.dbf
    - DISTRICT91.shp.xml
    - variables.xlsx
```
*Above: a sample directory structure for a SDR Shapefile or GeoTIFF layer submission. That is, each folder must contain all of the discrete files UNZIPPED that are part of a Shapefile, and they can contain other stuff, like XML metadata, homegrown codebooks, etc.*

The same can be said for the codebook or documentation folder that corresponds with each layer. For example:

```
- nyu_2451_12345_doc/
    - variables.xlsx
    - codebook.txt
    - DISTRICT91.shp.xml
```
*Above: a sample directory structure for the documentation folder. Note that it could be good practice to put standard metadata files in both the primary download folder and the documentation folder*

There are a litany of ways of outputting files that make it more or less easy to place them within the appropriate folder and file structure. Some of these may involve homegrown scripts, but in either case, even if you're dragging files into a pre-fabricated list of folders, you're coming out ahead. And the containers will help you to stay organized.

**Suggested advice:** the step of organizing your files can take a lot of time, so when you're done, you may want to preserve the folders and files you've organized in a safe place, because you will need them later to convert the shapefiles to SQL files. For example, it could be a good idea to make a copy of the files and folder with the data in it and put it somewhere else or drag it into the cloud just in case.

After all of the files are in place in the appropriate folder, the final part of this process is to run an **SdrFriend** command to zip individual files up into an archive named after the containing folder:

```bash
bundle exec rake files:zip_bitstreams[/Users/andrewbattista/Desktop/containers_for_collection]

## In this command, stipulate the folder path where the `containers_for_collection` folder is

```
At this point, each folder structure should now look like this:

```
- nyu_2451_12345/
  - nyu_2451_12345.zip
- nyu_2451_12346/
  - nyu_2451_12346.zip
- nyu_2451_12347/
  -nyu_2451_12347.zip
```
After the zips have been created successfully, you might want to open a few of them and spot check the files to make sure they are all there and are all created accurately. Again, leave the new data you've just zipped up alone and make a copy of it to inspect. If everything looks good, you can move on to the next step of uploading basic metadata to the FDA.

## 5. Upload basic descriptive metadata to FDA

Before all of the primary data bitstreams and documentation bitstreams are uploaded, it is a good idea to push our agreed-upon rudimentary descriptive metadata to the FDA. In order to do this, we will use the **SdrFriend** GeoBlacklight-to-FDA metadata command:

```bash
bundle exec rake fda:gbl_to_fda_metadata[/Users/andrewbattista/Downloads/UAE_metadata_final.csv]

## Here, the CSV input should be the .CSV file where your partially completed metadata records exist. You should just use the same CSV file you have been working on and downloaded earlier, provided all items have finalized titles and descriptions (at least).
```
Once you run this command, check out a few items in the FDA to see if the titles and descriptions loaded correctly. The upside of doing this step now is that it makes it easier to check on the accuracy of bitstreams once we upload them (since you'll be able to see the names of the items in the FDA). Note that you should *never* edit metadata directly in the FDA, even if you catch a typo or something. All metadata edits should be reflected in the canonical metadata copy in OpenGeoMetadata, which means for now that you should make the edit(s) you need in the Google Sheet. Periodically, we will use the `fda:gbl_to_fda_metadata` command to make sure the FDA basic metadata is in sync with what appears in GeoBlacklight.

## 6a. Upload bitstreams to FDA

Now that the basic metadata is in place in the FDA, we need to actually upload the various primary data & codebook bitstreams (when applicable) to their respective FDA records. We can do this either one bitstream at a time, or in batch:

```bash
bundle exec rake fda:addbit[/Users/sgb334/Desktop/nyu_2451_12345.zip]

## This command adds a single bitstream to a single record
```

The command above will upload the referenced bitstream to the appropriate FDA record (i.e., the record at http://hdl.handle.net/2451/12345). This will only work, however, if the bitstream follows the naming convention, and if the Handle lookup table has an entry corresponding to this Handle. Otherwise, we would need to specify exactly where we want to upload the bitstream to by referencing that record's internal dspace_id:

```bash
bundle exec rake fda:addbit[/Users/sgb334/Desktop/my_file.zip,53423]
## The number after the file is the dspace_id of the item you're uploading to
```

Alternatively, if we are uploading a large number of files that are all stored in the same directory, we can use **SdrFriend** to batch upload them. For this to work, we **must** have files that follow the naming convention detailed in section 4.

```bash
bundle exec rake fda:bit_batch[/Users/sgb334/Desktop/containers_for_collection,zip_only]

## In this command, the first part points to the path on your computer where the completed bitstream container is, and the second part is a parameter that allows you to stipulate "upload zipped items only." It's best practice to add this in at this step.
```
Running this command requires some monitoring. The bitstream uploads may trip up, but the report on the console will tell you which ones have uploaded successfully and which ones have not. When you get an error, re-run the command. Before re-running this command, though, you'll need to go in and "re-create" the container folders and take out all of the bitstreams that have already successfully uploaded, or else the command will throw an error again. The best way to do this is simply to create a new folder called `containers_for_collection_subset` or something like that and drag in any of the files from the original `containers_for_collection` folder that didn't yet upload. Or, just delete the files that have been successfully uploaded (provided that you previously made a copy of all of the data elsewhere). Also, spot check each of the uploads in the FDA and download a few of them to verify that the zips have been created successfully.

## 6b. Retrieve the bitstream URLs to plug into our metadata records

Now that we have uploaded all of the bitstreams to the FDA successfully, we can use **SdrFriend** to retrieve the existing bitstream download URLs, which are important elements for GeoBlacklight metadata. In order to do this, run the following command:

```bash
bundle exec rake metadata:bithydrate_csv[/Users/andrewbattista/Downloads/UAE_metadata_final.csv,/Users/andrewbattista/Downloads/UAE_collection_bitstreams.csv]
# The first part should be a .CSV file that you have. Given the logic of the workflow, it will likely be the same .CSV that you've been using throughout this process to create FDA metadata, GeoBlacklight metadata, etc. The second part of this command generates a new CSV (here you tell it where you want the new file to be)
```

After running the command, open up the newly created bitstreams CSV with Atom or Excel, copy the bitstream URLs, and stick them back into the main metadata CSV template in Google Sheets that you have been working on. Now that you have done this, you have all of the required elements for finalized GeoBlacklight metadata (assuming that you have also taken care of the Solr bounding boxes; if you haven't gotten the bounding boxes yet, now is a good time to do it).

## 7. Finalize GeoBlacklight Metadata Records and export them as .JSON

Now that you have a completely filled out CSV, use **SdrFriend** to transform it into a single JSON file. First, download or save the file you're working on as a CSV (again, remembering to avoid numbers or spaces in the filename). Then, run the following command:

```bash
bundle exec rake metadata:csv_to_json[/Users/sgb334/Downloads/eastview_files.csv,/Users/sgb334/Downloads/eastview_files_singlefile.json]

# First part: Path and location where your CSV is
# Second part: Name of your new .JSON file that has all of the individual layer records within a single file.
```
Doing this will generate a single file with as many .JSON records as there are rows in your CSV. It's a good idea to save this file for the subsequent step of indexing into Solr. For now, though, the next step is to split this one .JSON record into many individual item records that are named `geoblacklight.json` and reside in folders named according to our handle naming structure convention with OpenGeoMetadata. In order to do this, run the following script from the command line in Ruby (begin by typing in `irb`):

```bash
require 'json'

irb_context.echo = false

nyu_file = File.read('/Users/andrewbattista/Downloads/UAE_complete_metadata_singlefile.json')

## The file where the original .json that contains all of the records is above. Make sure to include the full path. Change accordingly

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
At this time, **SdrFriend** does not have a native split function, but this may be developed for NYU's convention only; other institutions' repositories will have to be managed with an individual script. For now, this script will work to produce individual JSON files that adhere to NYU's file-folder naming convention. Once these folders and files are created, you're ready to move on to step 10, committing these finalized records to the OpenGeoMetadata repository (see below).

## 8a. Create SQL versions of datasets and upload to PostGIS

*Note: This step is only relevant for vector files. If you are accessioning raster images, such as GeoTIFFs, skip to step 8b.*

#### Overview of vector processing

To enable previews and downloads via GeoServer's WMS / WFS services, we have to create a SQL version of our Shapefiles (in the projection EPSG:4326), and add them to the PostGIS database which stores EPSG:4326 "preview" versions of every vector layer in our collection. Earlier, while assembling bitstream packages for upload to the FDA, the procedure was to simply package up the original geospatial data layer as we downloaded it or received it. Now, however, in order to connect to GeoServer we'll have to make some modifications to it, which include:

- Reprojecting the layer to the standardized Coordinate Reference System (CRS) `EPSG:4326` (if it isn't already in that CRS)
- Renaming the file to represent the Handle associated with it. Earlier we were just naming the containing folder using the Handle convention, and letting the Shapefile keep its original name; now we have to make sure the layer and all the data files that comprise it uses the Handle as its name, so that the resulting table on PostGIS is predictably named
- Creating a SQL version of the Shapefile. This involves converting a Shapefile into a SQL script that creates a new table (named according to the pattern `nyu_XXXX_XXXXX`)
- Connecting to the PostGIS database on AWS and insert the new layer using its SQL script

#### Setting up dependencies for vector processing

There are a few dependencies and "one time" things that you need to install or set up so that everything within the suite of vector processing scripts will work properly and smoothly. Before this process begins, you will need to have downloaded the .pem file from the NYU Box SDR credentials folder.

First, on a Mac that has HomeBrew already, install GDAL by running these commands from Terminal:

```bash
brew update
brew doctor

brew install postgresql ## psql
brew install gdal ## ogr2ogr
brew install postgis ## shp2pgsql
```
Another "meta step" is to create a "favorite" in your [Transmit FTP client](https://transmit.en.softonic.com/mac) so you can easily connect to the `metadata.geo.nyu.edu` server. It is recommended that you buy a copy of Transmit if you're going to work with this step of the collection process frequently.

You will need to SSH into the metadata server. Here is the command to do that:

```bash
ssh -i /Users/staff/Documents/SDR_Credentials/key-1-jun24-2015.pem ubuntu@metadata.geo.nyu.edu
## The full path is the location on your hard drive where the permissions file is which is then followed by the username@the-server, in this case ubuntu@metadata.geo.nyu.edu
```

After running the command, you may get a message to update the permissions of the key file. In order to do this, run [this one-time command](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)(it's step number 3):

```bash
chmod 400 /Users/staff/Documents/SDR_Credentials/key-1-jun24-2015.pem
## Again, this path should represent the location of the permissions file on your hard drive
```

You should only have to run this one time. Next, you open Transmit, tab over to "Favorites," and click the plus at the bottom to add a new favorite. You will need the username, which is `ubuntu`, and the password, which you can find via NYU's internal documents. Actually download the .pem file onto your hard drive, as per the above command would suggest, and then connect it when you log in.

#### Actually preparing the files for transformation within Transmit

Assuming that you have already made the requisite connections to the `metadata.geo.nyu.edu` server in Transmit, you're now ready to actually convert files.

It's likely that you have saved the container folders that you produced in step 4 above, but you may only have the zipped up bitstreams left. If that's the case, you'll need to run the following script to "unzip" the bitstreams but leave them in their containing folders:

```bash
require 'find'

FOLDER_WITH_CONTAINERS="/Users/andrewbattista/Desktop/sdr_collection"

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

After running this script, you have the files you need for the conversion into SQL. If you never did zip up the files, or you've preserved the unzipped shapefiles, you won't need to do the above script. Instead, you can move on and SSH into the `metadata.geo.nyu.edu` server:

```bash
ssh -i /Users/staff/Documents/SDR_Credentials/key-1-jun24-2015.pem ubuntu@metadata.geo.nyu.edu
## The full path is the location on your hard drive where the permissions file is which is then followed by the username@the-server, in this case ubuntu@metadata.geo.nyu.edu
```

Once you are connected to the `metadata.geo.nyu.edu server`, use Terminal to navigate to the vector-processing directory. The first thing you'll want to do is run the cleanup command to remove any existing files that might be occupying the folders already. Remember that to navigate around in Terminal hit `ls` to list the sub-folders in the directory and then `cd` to enter the folder you want. To run the cleanup command, enter the following:

```bash
bash cleanup.sh
```

Check inside the folders in Terminal to make sure that all of the files are cleaned out. After the files are cleaned out, you can now place your original, unprojected files into the folders so that the vector processing script will work. Simply take the original shapefiles and place them into the `input_shp_to_WGS84` folder.  Find the files from the bitstream package construction step (see step 4 above), make sure the files are unzipped elements that are inside of a folder that has the naming convention spelled out in step 4 and then drag and dump the folders with the files in them into the aforementioned `input_shp_to_WGS84` folder (see image below). Doing this is part one of the process (re-projecting original files into WGS84).

![Transmit view of vector-processing-script ](./images/transmit-metadata.geo.nyu.edu.png)

#### Run the `vector-processing-script/processing.sh` script to convert files

Now that the files are in place in Transmit, we are ready to run the conversion script. To run it, enter the following command in Terminal.

```bash
cd vector-processing-script
bash processing_May16_revise.sh
## This command reflects a version of the script that assumes the files you are converting had attributes encoded in UTF-8, which may not always be the case. If in doubt, look at the difference between the May 16 version of the script and the original processing.sh script, and run the original script if necessary.
```

After entering that command, the script should then run via an interactive menu that has a heading saying **NYU SDR Processing Script** Simply follow its prompts. Most of the steps are intuitive, but the key is that you will have to re-name each of the files one-by-one according to the nyu_XXXX_XXXXX convention when you get to the conversion process. Doing this takes concentration, but if you have the original CSV that has the metadata in it with the original filenames (and in the same order that you associated them with existing handles), you can stay on track.

After you have created new SQL files, they should be posted to the PostGIS database on their own. To check, open the `output_shp_to_sql` folder and check to see if the SQL files are in there.

## 8b. Upload raster images to GeoServer

*Note: If the collection items you are processing are Raster (GeoTIFFs), you don't need worry about the entire process described in 8a. Instead, follow this process. Better instructions are forthcoming*

Each GeoServer host (Public and Restricted) has access to a single EFS (Amazon NFS) share that is mounted on both hosts at `/efs`.

Raster layers are stored in `/efs/geoserver/raster`:

```bash
ubuntu@ip-172-31-48-183:~$ cd /efs/geoserver/raster
ubuntu@ip-172-31-48-183:/efs/geoserver/raster$ ls
nyu_2451_34189  nyu_2451_37668  nyu_2451_40801  nyu_2451_41076  nyu_2451_41351 ...

ubuntu@ip-172-31-48-183:/efs/geoserver/raster$ cd nyu_2451_34189
ubuntu@ip-172-31-48-183:/efs/geoserver/raster/nyu_2451_34189$ ls
nyu_2451_34189.tif
```

##  9. Enable layers on GeoServer

GeoServer can be interacted with through a web-interface, or a REST HTTP API. It is recommended to update or "turn on" layers in batch using the REST API. First, though, it helps to understand the architecture of NYU's GeoServers.

#### Fundamental layout of NYU's GeoServer implementation

We are maintaining two GeoServers:

- Maps-Public ([maps-public.geo.nyu.edu](http://maps-public.geo.nyu.edu))
 -- Layers published in this instance are accessible to everyone in the world
 -- This instance contains layers for all NYU hosted records with the `"dc_rights_s": "Public"` value
- Maps-Restricted ([maps-restricted.geo.nyu.edu](http://maps-restricted.geo.nyu.edu))
  -- Layers published in this instance should be accessible only to the NYU IP range, which is already defined in the application, and to other hosts in the AWS virtual cloud
  -- This instance contains layers for all NYU hosted records with the `"dc_rights_s": "Restricted"` value

Each GeoServer instance has access to map data in two ways:
- Vector
  -- Vector data is served through a database connection to our AWS-hosted PostGIS
- Raster
  -- Raster data is served using files stored on two EBS volumes (one for the Public host, one for the Restricted host), directly mounted on each respective GeoServer host at `/ebs`


#### Enabling layers via the GeoServer API

Enabling layers via the GeoServer API is more efficient. In order to do this, use the GeoServer rake task within the **SdrFriend** to enable the layers:

```bash
bundle exec rake geoserver:enable[Users/andrewbattista/UAE_collection_metadata.csv]
## In this command, the path stipulated is the CSV that contains all of the files used to process the collection. This should be the same CSV used to generate the metadata in step 7 above.
```
Once this command happens the layers will be "activated" in the appropriate instance of GeoServer. The blank line beneath each layer that gets posted is the response from the server, and no response is what you want. After the layers have loaded, you can log in to the web interface of the respective GeoServer instance (public or private), click on layer previews, search for a layer from the newly created files, and verify that they are loading and previewing correctly. You may want to examine the attribute table and the geometries in tandem with downloading the original file in QGIS.

## 10. Commit records to the OpenGeoMetadata repository

Now we return to the metadata. Once you have finalized your metadata records (see step 7), the next step is to commit them to the [`edu.nyu` repository](https://github.com/OpenGeoMetadata/edu.nyu). You can do this as soon as you finish step 7 and place it here, or you can wait until you have finished steps 8-9. It doesn't matter. First, take the newly created folders and files from step 7 and create a new update branch on the `edu.nyu` master repository. The best practice is to commit your changes to that new collection branch, then issue a pull request from that branch into the master branch.

There are two ways to create a new branch and submit it as a pull request: via Git commands or via the GitHub desktop software. Here's the sample workflow with Git:

```bash
cd ~/git/edu.nyu
git checkout master
git pull

## This initial command makes sure you have downloaded the most current version of the edu.nyu metadata repository

## Switch to a new local branch, based on master
git checkout -b my_new_collection
## -b creates a new branch called 'my_new_collection'
```

At this point, add all of the new records to the `~/git/edu.nyu` directory. You can simply drag the folders in to their respective location on your own hard drive. Once the new files are in place, commit and push to GitHub with the sequence below:

```bash
git status

## just to inspect the changes, which in this case will be the newly added files
```
Then:

```bash
git add . ## Stages all changes
git commit -m "Adds records for whatever collection"

## Push the commit to a new branch on Github
git push --set-upstream origin my_new_collection
```

Now the new records have been posted to Github. If you go to the [main page for the repository](https://github.com/OpenGeoMetadata/edu.nyu), you should be able to see that a new branch (`my_new_collection`) was just pushed. Take a look at the branch, and if everything seems good, issue a pull request and assign another member of the geo team to review it. **It's typically considered poor form to approve your own pull request, so assign it to another member of the geo team to review** If you have been assigned to review, the things you should look for include: making sure the file-folder convention match the identifier for the record, making sure no needless blank elements exist, making sure all elements are filled out correctly, etc. If all is good, you can both approve the pull request and merge into the master branch with the command below:

```bash
git checkout master
git pull
```

You can also review and confirm the merge on the GitHub web interface. After that pull request is approved and merged into the Master branch, make sure to delete the `my_new_collection` branch on Github, since it is no longer needed, and then delete it on your local machine. The new metadata records are now complete an in the collection.

The final part of this process is doing one more check to ensure that all of the records are GeoBlacklight compliant. A Travis continuous integration (CI) script on GitHub should kick off for any commit to the repository. It will attempt to validate every record (using [GeoCombine](https://github.com/OpenGeoMetadata/GeoCombine)), and log the results. Note that the build typically takes a few minutes to complete. If there are any errors, you can come up with a strategy for fixing them.

You can also use the GitHub desktop software to accomplish this same process. In order to do that, first clone the entire `edu.nyu` repository to your hard drive, then drag in new the new metadata files you have created. If you haven't done so already, add the repository to your application by clicking the plus button at the top left. Immediately, you will see differences of the changed files that have been added or edited. Next, create a new branch, and name it according to the collection. Finally, commit that branch to the master and then navigate to the web interface to issue the pull request.

## 11. Index newly updated metadata records into Solr

#### Installing a local development instance of the NYU Libraries GeoBlacklight repository

Now that the new metadata records are a part of the master branch of the `edu.nyu` repository, we are ready to index them into Solr. The first thing to do is index them into a development instance of Solr and GeoBlacklight to make sure that all of the records look okay and don't cause any errors. If you haven't done this before, you'd first need to make a clone of the NYU production GeoBlacklight on your computer. The repository is located [here](https://github.com/NYULibraries/spatial_data_repository). We will need to put in the .yml file that contains the secret passwords. That is kept in the credentials box drive and is called `geoblacklight.yml`. Copy that file, drag it into spatial_data_repository/config, and then rename it to `vars.yml`

#### Installing Solr and other requirements

The other prefatory step you need to take is to set up Solr and the GeoBlacklight Solr core in the dev version on the local hard drive. To install Solr, type in the following command:

```bash
brew install solr
```
Once it's installed successfully, go to Terminal and type:

```bash

solr start

## solr stop stops Solr and solr status lets you know if it's running and solr restart restarts
```

Now, go to [this address](http://localhost:8983/solr/). Once you get there, you've arrived at the web interface of Solr. Now you need to make sure there is valid GeoBlacklight core. To do this, download the GeoBlacklight core zip that is in the NYU box drive. Then go to your local install of Solr, specifically this directory: `/usr/local/Cellar/solr/7.3.1/server/solr` and put the unzipped core into it. If you can't find this folder on your Mac, hit `command+shift+g` and enter the path to the aforementioned directory. Your path might be slightly different depending on which computer you are using. Once you've added the unzipped core, you need to re-start Solr. Now, go back to the Solr link in your browser, go to the main console and make sure you see a Blacklight core on the core selector pulldown menu. Select it to activate it.

Now we can start up the development version of Solr. First, navigate to the directory where the files for GeoBlacklight exist on your local machine and type `bundle` to make sure all prerequisites are installed. There seem to be errors here at this point that have to do with having the right version of mysql2 installed. If you're able to have all of the prerequisites installed, you can move on to starting up the Rails server with this command:

```bash
bundle exec rails s -b 0.0.0.0
```
Now the instance is started. Leave the window open. And go to http://localhost:3000. You're now there.

#### Indexing into development

Assuming that you have already installed Solr, we can test index newly created GeoBlacklight records to see if we want them to be in our environment. First, start Solr:

```bash
solr start
```

Next, navigate to the Spatial Data Repository folder and start the Rails server. That could be something like:

```bash
cd ~/git/spatial_data_repository
## this is the sample path. It could be different if you have it installed elsewhere
bundle exec rails s -b 0.0.0.0
```
Now that the Rails server and Solr are running on your local machine, confirm that it's working correctly. Go to [this link](http://localhost:3000/) to check. If it all is working you'll see a GeoBlacklight. Also, make sure that the Terminal window you used to launch this stays open, or else the development instance will shut down.

Now, we will index the new GeoBlacklight records into our development Solr core. To do this, clone the most current version of the `edu.nyu` metadata repository on your machine and then place it in the `GBL-metadata-development` folder that you have created on your hard drive. You may also want to place other new GeoBlacklight files that you have created into the `GBL-metadata-development` folder. All that matters is that there are documents within this folder that end in `geoblacklight.json`.

The final step is to add in a valid `index-records.rb` script into the folder. We have these sitting on our production instance right now, but for reference, you can also download the script in the [snippets part of the SDR documentation repository.](https://github.com/sgbalogh/sdr-documentation/blob/master/ruby-scripts-for-metadata.md) **Note that you will have to copy this file to a text editor, modify the variables to match the place(s) on your hard drive, and then save the file within the `GBL-metadata-development` folder on your hard drive.**

Once you've saved a version in the right place, navigate to the right place within Terminal and run this command:

```
ruby index-records.rb
```
 
The script indexes new records. When it's done, go back to your web version of the development GeoBlacklight. You should be able to see the new records and verify that they work. Search for one of them. If all looks good and there are no damning errors, you're ready to index into production. Make sure you click on as many elements of records as you can. Try doing spatial searching, making faceted searches, etc. Once you're ready to go live, make sure that any new metadata items you're adding are part of the master branch of OpenGeoMetadata.

#### Indexing into production

Before running the script to index items into the production instance of Solr, you need to SSH into the `metadata.geo.nyu.edu` server. To do this run:

```bash
ssh -i /Users/staff/Documents/SDR_Credentials/key-1-jun24-2015.pem ubuntu@metadata.geo.nyu.edu
```

Once you're in this server, make sure that the metadata folder in that server is up to date with the master branch of `edu.nyu` on OpenGeoMetadata. To do this, `git pull` the most recent version of the NYU repository in the OpenGeoMetadata server. To do this, go to the `edu.nyu` directory within the `metadata.geo.nyu.edu` server that we've SSHd into.

Once you have all the most current metadata in place, use the script that is located within the folder on the server. You should only have to run this command:

```bash
ruby index-records.rb
```

That's it! If there are no errors, the records have been loaded. The collection workflow is complete and the records are live in GeoBlacklight, preserved within NYU Libraries, and available to the NYU community, as well as the metadata available to the larger GeoBlacklight community, thus fulfilling the first goal of the [NYU Libraries Strategic Plan.](https://sites.google.com/nyu.edu/stratplan/)

## Appendix: Additional and optional steps to augment the workflow

The process above is the essence of the collection workflow, but there are other miscellaneous commands and projects that can be used at various points. You will also want to check the snippets part of the SDR Documentation repository for additional code snippets and ideas that may be helpful.

### a. Calculating bounding boxes for `solr_geom` field

All GeoBlacklight records require a bounding box. There are a lot of ways to generate this data, but you will probably want to do that automatically if your collection has many kinds of shapefiles or GeoTIFFs that cover many different areas. GDAL is useful for this. **SdrFriend** provides a wrapper over GDAL, and a utility for generating `solr_geom` syntax bounding boxes, given a Shapefile. Note that the Shapefile should be in EPSG:4326 (WGS84) CRS before running the command, so you will have to batch convert the files to the aforementioned coordinate reference system if they aren't in it already. To locate the geometries for an individual file use this command:

```bash
rake gdal:bounding[/path/to/file.shp]
```

Or, alternatively, you can use **SdrFriend** for finding all Shapefiles, recursively, existing within a directory, and have the output printed out for all of them:

```bash
rake gdal:bounding_many[/path/to/shapefile_directory]
```

Once the values are generated, you can copy them and paste them back into the CSV or Google Sheet you have been using to make the GeoBlacklight metadata.

### b. Removing .DS_Store files

These files appear if you ever change the way Finder displays things, and they can creep into your repository. Normally you can't see these files, but git is able to see them, and you have to delete them when making commits. The link that explains this is here.

- [Removing .DS_Store files](https://jonbellah.com/articles/recursively-remove-ds-store)

To delete these files before committing anything to a repository, run this command
```
find . -name '.DS_Store' -type f -delete.
```

The files should be deleted. You can always delete the files manually in GitHub as well.

### c. Putting the elements within a batch of JSON records into alphabetical order

### d. Removing a single element from a batch of JSON records.

Depending on the nature of the collection upload, you may want to delete a single key from a set of metadata records. Example scenarios could include getting rid of the `nyu_addl_format_sm` element if you're cleaning records from other institutions (and using **SdrFriend** to do it), getting rid of the `dc_creator_sm` key if none of the records in that batch have a creator (for example), getting rid of a field that has been deprecated, etc. If this is what you want to do, you can use the following script:

```bash
require 'json'

collection = JSON.parse(File.read("/Users/staff/Downloads/GlobalMap_singlefile.json"))
 ## put the path to the file where your single file JSON is

collection.each do |record|
  record.delete("nyu_addl_format_sm")
  ## here, stipulate which field you want to delete
end

File.open("/Users/staff/Downloads/GlobalMap_singlefile.json", "w") do |f|
  f.write(JSON.pretty_generate(collection))
end
```
Note that in order for this script to work, your file should not have spaces or numbers in it. Otherwise you are good and we have created the collection.

### e. Updating records from another institutions

In order to update records from another institution, enter the production folder of the `metadata.geo.nyu.edu` server, then enter the folder of the repository you want to update and then hit git pull. This updates the files and adds the NYU specific fields to our production instance. Then, re-run the `index-records.rb` script (see above). That's all you have to do.

### f. Using the `vector-processing-script` tool for actual data conversion

Now you're ready to actually convert the files into SQL tables. If you're reading this documentation, you should also have access to a script located in a [repository called `vector-processing-script`](https://github.com/sgbalogh/sdr-vector-processing). This script provides a simple command-line interface to some common data-processing steps. It is essentially just a wrapper on top of GDAL / OGR commands.

#### Examples of main commands

If the script isn't working properly, or if you'd prefer some additional control over the conversion-and-upload-to-PostGIS process, these are the main commands to consider:

```bash
## Reproject a Shapefile into EPSG:4326 (WGS84)
ogr2ogr -t_srs EPSG:4326 -lco ENCODING=UTF-8 output.shp input.shp
#ogr2ogr -t_srs EPSG:4326 -lco ENCODING=UTF-8 nyu_2451_12345_WGS84.shp NYCSchools.shp

## Convert an EPSG:4326 Shapefile into SQL
shp2pgsql -I -s 4326 input.shp input > output.sql
#shp2pgsql -I -s 4326 nyu_2451_12345_WGS84.shp nyu_2451_12345 > nyu_2451_12345.sql

## Upload the SQL file to PostGIS
psql --host nyu-geospatial.cfh3iwfzn4xy.us-east-1.rds.amazonaws.com \
     --port 5432 \
     --username sdr_admin \
     --dbname geospatial_data \
     -w -f output.sql
```
If you are ever trying to connect to the SQL database from a different machine, you may run into restrictions. In order to run the last command, which connects to PostGIS and attempts to insert the contents of `output.sql` as a new table, you will also need to supply the password for the database (and make sure there are no firewall impediments to you connecting directly to the database). See the credentials documentation for more info.
