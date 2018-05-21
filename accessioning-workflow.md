# SDR Collection / Acquisitions Workflow
Stephen Balogh <sgb334@nyu.edu> & Andrew Battista <ab6137@nyu.edu>

### Note

Many of the examples in this document make use of a Ruby gem called **SdrFriend**, so named because of how friendly it is. :smiley:
The codebase for that tool, as well as documentation (including on how to set it up), can be found at its [Github page](https://github.com/sgbalogh/sdrfriend).

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
The above process is just one way to determine how many layers exist in a collection. You could also count these manually, or view the list of files in Finder, select them all, then hit ``command+c`` and then paste the result into a spreadsheet.

## 2a. Creating Repository Records / "Minting Handles"

Once the number of items in a given collection is determined, the next step is to "mint" the requisite number of IDs from the FDA. SDR assets span two different collections on the FDA, both located within the [Division of Libraries]("https://archive.nyu.edu/handle/2451/14821") community. They are:
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
If the above command is successful and no errors were thrown, you should have saved a CSV text file in the format  "*handle*,*dspace-internal-id*". The recommended approach to updating the lookup table is to open this new file, copy the handles and DSpace ids, and then paste them into your locally held, checked out version of the `/misc/handle-dspace-lookup.csv` file on your hard drive (see instructions on this in the comments below).

*Additional note: for each of these commands, spacing and capitalization matters, so it's easiest to copy them directly from this documentation.*

## 2b. Updating the Handle table

The FDA (confusingly) uses two types of IDs to refer to records: an internal id (referred to as "dspace_id" in this document), and a Handle identifier. If only these were the same!

The Handles are great for public consumption, but unfortunately the REST API for the Faculty Digital Archive (i.e., the way in which programmed tools interact with the repository) requires the internal dspace_id, and there's no simple way to turn one into another. As a work around, we manage our own lookup table, stored in the [edu.nyu repository]("https://github.com/OpenGeoMetadata/edu.nyu/blob/master/misc/handle-dspace-lookup.csv") on GitHub.

**Make sure to update this table whenever you pre-allocate / "mint" new FDA records.**

One way to make updates to the lookup table is to set up git to commit to the edu.nyu metadata repository (see [these instructions on how to do it](https://help.github.com/articles/set-up-git/)). If you're already set up to commit to the edu.nyu repository, here's a simple way to do that:

```bash
cd ~/git/edu.nyu
git pull
git checkout master

## Now we can append the CSV file of new Handles to the end of the lookup table.
## To do this, the easiest way is copy the direct output (file) of the handles and DSpace IDs as it appears in Terminal.
## Then, go to the lookup table folder within the misc folder in the edu.nyu repository on your drive, open the .CSV with Atom, and paste the new handles in there. Make sure there are no extra lines at the end.
## Then, run this next command:

cat ~/Desktop/new_handles.csv >> ./misc/handle-dspace-lookup.csv

## Make sure that the above command reflects the actual name of the new .csv you created. After you run the command, nothing visible "happens" on the command line.
## Inspect that everything worked by executing the following commands:

git status

git diff misc/handle-dspace-lookup.csv

## If it looks good, go ahead and commit it
git add ./misc/handle-dspace-lookup.csv ## stage the changes in the file
git commit -m "adds some new Handles for X collection"
git push
```
*Note: if any of the above looks confusing to you, there are a million other ways to update the table: you could try using the GitHub Desktop app, for instance. You could manually concatenate the two CSV documents using Excel (or even just a text editor like Atom).*

## 3. Create GeoBlacklight OpenGeoMetadata

At this point, you've got a grasp on what documents exist and need to be described, and you've already pre-allocated the appropriate number of FDA records in the relevant collection. Now you'll want to start actually constructing what will end up being the various GeoBlacklight records describing your layers.

These days, it seems like simply working in a spreadsheet is the best way to start. Begin by making a copy of the [GeoBlacklight template](https://docs.google.com/spreadsheets/d/1XAjL59fEsegZm2dArVjeT0XaOLohvDIkSQ9_RftSPlI/edit?usp=sharing). Alternatively, you can have **SdrFriend** generate one for you:

```bash
cd ~/git/SdrFriend
bundle exec rake metadata:template[/Users/sgb334/Desktop/new_collection.csv]
## this stipulates the path where you want to save the file and the name of the file
```
The first data you should add is the dspace_id and Handle attributes from the collection you pre-allocated. Go ahead and paste them into the CSV. Feel free to add in additional columns to the CSV document if it's convenient to do so (such as "original file names"), though **only the columns listed in the template** will persist after converting the CSV to actual GeoBlacklight records.

Also, very important: **don't rename any of the existing template columns.**

From here, do whatever you need to do to fill in the rest of the columns. In general, you are responsible for all of the metadata elements; the only ones which cannot be filled in at this time are the `ref:download-url` and `ref:documentation-url` fields, which are intended to point to FDA bitstreams. That workflow is described below. If you are using the Google Sheets template, the `layer_modified_dt` field should also automatically update as you work in the Sheet. If you have any questions about the rules or best practices for filling out metadata fields, refer to the [GeoBlacklight Schema](https://github.com/geoblacklight/geoblacklight/blob/master/schema/geoblacklight-schema.md). One of the most difficult fields to come up with is the Solr geom, so refer to the appendix of this document for an **SdrFriend** command to find the `solr_geom` of a given file.

Every FDA/GeoBlacklight layer needs at least one download url (the main, `ref:download-url` value). If your layer comes with additional documentation or codebook material, you may also want to link directly to that in the GeoBlacklight record (via `ref:documentation-download`), and make use of GeoBlacklight's contextual rendering for it. Unfortunately, though, it isn't possible to predict what the bitstream URLs will be until you have already uploaded them to your empty FDA container records. Thus, we will need to move on once we have enough metadata to otherwise constitute minimal viable GeoBlacklight records. Later we will insert the correct download URLs before outputting the "OpenGeoMetadata-ready" version of the records.

## 4. Create bitstreams packages and prepare them

At this point, we now need prepare to upload our content to the digital repository (FDA). We've already pre-allocated as many records as we'll need. First, we need to assemble the bitstreams that we will be uploading. Although it isn't perfect, the best strategy is to create a "container" for our impending upload. To do this, run the following task from *SDRFriend*

```bash
bundle exec rake files:download_containers[/Users/andrewbattista/Desktop/containers_for_collection,Users/andrewbattista/Downloads/UAE_collection_metadata.csv]

## In the above command, you are first stipulating the area of your computer where you want to create the new folders and then the second part is the name of the CSV that you're using (from step 3 above) to make the GeoBlacklight metadata. Likely, you will have downloaded this from your Google Sheet.

## Note also that this command just creates a structure of blank folders that are named according to the handles you've already created. After running this command, you'll need to do a similar thing to create documentation folders:

bundle exec rake files:documentation_containers[/Users/andrewbattista/Desktop/doc_containers_for_collection,Users/andrewbattista/Downloads/UAE_collection_metadata.csv]

```

After creating both sets of new folders, you may want to combine them into a single folder, or you may want to keep them separate. This is an assessment you'll have to make depending on the condition of the data you're taking in. Which leads us into the next point.

All SDR submissions have to be in a directory structure that looks like this (for example):

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
After the zips have been created successfully, you might want to open a few of them and spot check the files to make sure they are all there and are all created accurately.

## 5. Upload basic descriptive metadata to FDA

Before all of the primary data bitstreams and documentation bitstreams have been uploaded to the FDA, we need to push our agreed upon rudimentary descriptive metadata to the FDA. In order to do this, we will use the **SdrFriend** GeoBlacklight-to-FDA metadata command:

```bash
bundle exec rake fda:gbl_to_fda_metadata[/Users/andrewbattista/Downloads/UAE_metadata_final.csv]

## Here, the CSV input should be the .CSV file where your partially completed metadata records exist. You should just use the same file you have been working on, provided all items have finalized titles and descriptions (at least).
```
Once you run this command, check out a few items in the FDA to see if the titles and descriptions loaded correctly. The upside of doing this step now is that it makes it easier to check on the accuracy of bitstreams once we upload them (since you'll be able to see the names of the items in the FDA).

## 6a. Upload bitstreams to FDA

Now that the basic metadata is in place in the FDA, we need to actually upload the various primary data & codebook bitstreams (when applicable) to their respective FDA records.

We can do this either one bitstream at a time, or in batch:

```bash
bundle exec rake fda:addbit[/Users/sgb334/Desktop/nyu_2451_12345.zip]

## This command adds a single bitstream to a single record
```

The command above will upload the referenced bitstream to the appropriate FDA record (i.e., the record at http://hdl.handle.net/2451/12345). This will only work, however, if the bitstream follows the naming convention, and if the Handle lookup table has an entry corresponding to this Handle. Otherwise, we would need to specify exactly where we want to upload the bitstream to, by referencing that record's internal dspace_id:

```bash
bundle exec rake fda:addbit[/Users/sgb334/Desktop/my_file.zip,53423]
```

Alternatively, if we are uploading a large number of files that are all stored in the same directory, we can use **SdrFriend** to batch upload all of them. For this to work, we **must** have files that follow the naming convention detailed in section 4.

```bash
bundle exec rake fda:bit_batch[/Users/sgb334/Desktop/containers_for_collection,zip_only]

## In this command, the first part points to the path on your computer where the completed bitstream container is, and the second part is a parameter that allows you to stipulate "upload zipped items only." It's best practice to add this in at this step.
```
Running this command requires some monitoring. The bitstream uploads may trip up, but the report on the console will tell you which ones have uploaded successfully and which ones have not. When you get an error, re-run the command. Before re-running this command, though, you'll need to go in and "re-create" the container folder and take out all of the bitsreams that have already successfully uploaded, or else the command will throw an error again. The best way to do this is simply to create a new folder called `containers_for_collection_subset` or something like that and drag in any of the files from the original `containers_for_collection` folder that didn't yet upload. Also, spot check each of the uploads in the FDA and download a few of them to verify that the zips have been created successfully.

## 6b. Retrieve the bitstream URLs to plug into our metadata records

Now that we have uploaded all of the bitstreams to the FDA successfully, we can use **SdrFriend** to retrieve the existing bitstream download URLs, which are important elements for GeoBlacklight metadata. In order to do this, run the following command:
```bash
bundle exec rake metadata:bithydrate[/Users/andrewbattista/Downloads/UAE_metadata_final.csv,/Users/andrewbattista/Downloads/UAE_collection_bitstreams.csv]
# The first part should be a .CSV file that you have. Given the logic of the workflow, it will likely be the same .CSV that you've been using throughout this process to create FDA metadata, GeoBlacklight metadata, etc. The second part of this command generates a new CSV (here you tell it where you want the new file to be)
```
After running the command, open up the newly created bitstreams CSV with Atom, copy the bitstream URLs, and stick them back into the main metadata CSV template in Google Sheets that you have been working on. Now that you have done this, you have all of the required elements for finalized GeoBlacklight metadata (assuming that you have also taken care of the Solr bounding boxes).

## 7. Finalize GeoBlacklight Metadata Records and Export as .JSON

Now that you have a completely filled out CSV, use **SdrFriend** to transform it into a single JSON file. First, download or save the file you're working on as a CSV. Then, run the following command:

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
At this time, **SdrFriend** does not have a native split function, but this may be developed. For now, this script will work to produce individual JSON files that adhere to NYU's file-folder naming convention. Once these folders and files are created, you're ready to move on to step 10, committing these finalized records to the OpenGeoMetadata repository (see below).

## 8a. Create SQL versions of datasets and upload to PostGIS

*Note: This step is only relevant for vector files. If you are accessioning raster images, such as GeoTIFFs, skip to step 8b.*

#### Overview of vector processing

To enable previews and downloads via GeoServer's WMS / WFS services, we have to create a SQL version of our Shapefiles (in the projection EPSG:4326), and add them to the PostGIS database which stores EPSG:4326 "preview" versions of every vector layer in our collection. Earlier, while assembling bitstream packages for upload to the FDA, the procedure was to simply package up the original geospatial data layer as we downloaded it or received it. Now, however, in order to connect to GeoServer we'll have to make some modifications to it, which include:

- Reprojecting the layer to the standardized Coordinate Reference System (CRS) `EPSG:4326` (if it isn't already in that CRS)
- Renaming the file to represent the Handle associated with it. Earlier we were just naming the containing folder using the Handle convention, and letting the Shapefile keep its original name; now we have to make sure the layer and all the data files that comprise it uses the Handle as its name, so that the resulting table on PostGIS is predictably named
- Creating a SQL version of the Shapefile. This involves converting a Shapefile into a SQL script that creates a new table (with name in the pattern `nyu_XXXX_XXXXX`)
- Connecting to the PostGIS database on AWS and insert the new layer using its SQL script

There is only a single PostGIS database, and it is connected to both the Public and Restricted GeoServer Virtual Machines (VMs). Note that the distinction between "public" and "restricted" is not represented anywhere at the PostGIS level; this is fine, because our PostGIS database is not for consumption by any user other than GeoServer.

#### Setting up dependencies for vector processing

There are a few dependencies and "one time" things that you need to install or set up so that everything within the suite of vector processing scripts will work properly and smoothly. First, on a Mac that has HomeBrew already, install GDAL by running these commands from Terminal:

```bash
brew update
brew doctor

brew install postgresql ## psql
brew install gdal ## ogr2ogr
brew install postgis ## shp2pgsql
```
Another "meta step" is to create a "favorite" in your [Transmit FTP client](https://transmit.en.softonic.com/mac) so you can easily connect to the `metadata.geo.nyu.edu` server. First, you will need to SSH into the metadata server. Here is the command:

```bash
ssh -i /Users/staff/Documents/SDR_Credentials/key-1-jun24-2015.pem ubuntu@metadata.geo.nyu.edu
## The full path is the location on your hard drive where the permissions file is and then the username@the-server
```

After running the command, you may get a message to update the permissions of the key file. In order to do this, run [this one-time command](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)(it's step number 3):

```bash
chmod 400 /Users/staff/Documents/SDR_Credentials/key-1-jun24-2015.pem
```

You should only have to run this one time. Next, you open Transmit, tab over to "Favorites," and click the plus at the bottom to add a new favorite. You will need the username, which is `ubuntu`, and the password, which you can find via NYU's internal documents. Actually download the .pem file onto your hard drive, as per the above command would suggest.

#### Preparing the files for transformation within Transmit####

Once you are connected to the `metadata.geo.nyu.edu server`, use Terminal to navigate to the vector-processing directory and then use the `input_shp_to_WGS84` folder. Simply find the files from the bitstream package construction step (see step 4 above), but make sure the files are unzipped, and then drag and dump the folders with the files in them into the aforementioned `input_shp_to_WGS84` folder (see image below).

![Transmit view of vector-processing-script ] (./images/transmit-metadata.geo.nyu.edu.png)

You'll need to make sure that there's no data in the folder already. If there is, use the `clean` command to remove them all first.

#### Using the `vector-processing-script` tool for actual data conversion

Now you're ready to actually convert the files into SQL tables. If you're reading this documentation, you should also have access to a script located in a [repository called `vector-processing-script`](https://github.com/sgbalogh/sdr-vector-processing). This script provides a simple command-line interface to some common data-processing steps. It is essentially just a wrapper on top of GDAL / OGR commands.


#### Run `vector-processing-script/processing.sh`

The script implements workflows for the following conversions:

- SHP => reprojected SHP in `EPSG:4326`
- `EPSG:4326` SHP => SQL
- SQL => PostGIS insert statement

```bash
cd vector-processing-script
bash processing.sh
```

The script should then run via an interactive menu.

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

In order to run the last command, which connects to PostGIS and attempts to insert the contents of `output.sql` as a new table, you will also need to supply the password for the database (and make sure there are no firewall impediments to you connecting directly to the database). See the credentials documentation for more info.

## 8b. Upload raster images to GeoServer

*Note: If the collection items you are processing are Raster (GeoTIFFs), you don't need worry about the entire process described in 8a. Instead, follow this process.*

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

GeoServer can be interacted with through a web-interface, or a REST HTTP API. We will document both methods below, but it is recommended to update layers in batch using the REST API. First, though, it helps to understand the fundamental architecture of NYU's GeoServers.

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

#### Enabling layers via web interface

If you are only intending to publish one or two layers at a time, it might be a good idea to log on to the web interface for the appropriate GeoServer and do it manually. to do this, go to the [Maps-Public] (http://maps-public.geo.nyu.edu) interface, click layer previews, and log in (the username and password are held internally).

#### Via API

Enabling layers via the API is more efficient. In order to do this, use the GeoServer rake task within the **SdrFriend** to enable the layers:

```bash
bundle exec rake geoserver:enable[Users/andrewbattista/UAE_collection_metadata.csv]
## In this command, the path stipulated is the CSV that contains all of the files used to process the collection. This should be the same CSV used to generate the metadata in step 7 above.
```
Once this command happens the layers will be "activated" in GeoServer. The blank line beneath each layer that gets posted is the response from the server, and no response is what you want.

## 10. Commit records to the OpenGeoMetadata repository

Now we return to the metadata. Once you have finalized your metadata records (see step 7), the next step is to commit them to the [`edu.nyu` repository](https://github.com/OpenGeoMetadata/edu.nyu). First, take the newly created folders and files from step 7 and create a new update branch on the edu.nyu master repository. The best practice is then to commit your changes to that new collection branch, then issue a pull request from that branch into the master branch.

There are two ways to do this: via Git commands or via the GitHub desktop software. Here's the sample workflow with Git:

```bash
cd ~/git/edu.nyu
git checkout master
git pull

## This initial command makes sure you have downloaded the most current edu.nyu records

## Switch to a new local branch, based on master
git checkout -b my_new_collection
## -b creates a new branch called 'my_new_collection'
```

At this point, add all of the new records to the `~/git/edu.nyu` directory. You can simply drag them in. Once the new files are in place, commit and push to GitHub.

```bash
git status

## just to inspect the changes
```
Then:

```bash
git add . ## Stages all changes
git commit -m "Adds records for whatever collection"

## Push the commit to a new branch on Github
git push --set-upstream origin my_new_collection
```

Now the new records have been posted to Github. If you go to the [main page for the repository](https://github.com/OpenGeoMetadata/edu.nyu), you should be able to see that a new branch (`my_new_collection`) was just pushed. Take a look at the branch, and if everything seems good, issue a pull request and assign another member of the geo team to review it. **It's typically considered poor form to approve your own pull request, so assign it to another member of the geo team**

After that pull request is approved and merged into the Master branch, make sure to delete the `my_new_collection` branch on Github, since it is no longer needed, and then delete it on your local machine:

```bash
git checkout master
git pull
```

The final part of this process is doing one more check to ensure that all of the records are GeoBlacklight compliant. A Travis continuous integration (CI) script on GitHub should kick off for any commit to the repository. It will attempt to validate every record (using [GeoCombine](https://github.com/OpenGeoMetadata/GeoCombine)), and log the results. If there are any errors, you can come up with a strategy for fixing them.

You can also use the GitHub desktop software to accomplish this same process. In order to do that, first clone the entire `edu.nyu` repository to your hard drive, then drag in new the new metadata files you have created. If you haven't done so already, add the repository to your application by clicking the plus button at the top left. Immediately, you will see differences of the changed files. Next, create a new branch, and name it according to the collection. Finally, commit that branch to the master and then navigate to the web interface to issue the pull request.

## 11. Index newly updated metadata records into Solr

Now that the new metadata records are a part of the master branch of the `edu.nyu` repository, you are ready to index them into Solr. The first thing to do is index them into a development instance of Solr and GeoBlacklight to make sure that all of the records look okay and don't cause any errors.

We will use GeoCombine to index the records.

That's it! The collection workflow is complete.

## Appendix: Additional and optional steps to augment the workflow

The process above is the essence of the collection workflow, but there are other miscellaneous commands and projects that can be used at various points during the process.

### a. Calculating bounding boxes for `solr_geom` field

All GeoBlacklight records require a bounding box. There are a lot of ways to generate this data, but you will probably want to do that automatically if your collection has many kinds of shapefiles or GeoTIFFs that cover many different areas. GDAL is useful for this. **SdrFriend** provides a wrapper over GDAL, and a utility for generating `solr_geom` syntax bounding boxes, given a Shapefile. Note that the Shapefile should be in EPSG:4326 (WGS 84) CRS.

```bash
rake gdal:bounding[/path/to/file.shp]
```

Or, alternatively, you can use a **SdrFriend** task for finding all Shapefiles, recursively, existing within a directory, and have the output printed out for all of them:

```bash
rake gdal:bounding_many[/path/to/shapefile_directory]
```

Once the values are generated, you can copy them and paste them back into the CSV or Google Sheet you have been using to make the GeoBlacklight metadata.

### b. Caching tile layers to allow for easier page loading

### c. Removing .DS_Store files

- [Removing .DS_Store files](https://jonbellah.com/articles/recursively-remove-ds-store)

### d. Putting the elements within a batch of JSON records into alphabetical order

### e. Removing a single element from a batch of JSON records.
