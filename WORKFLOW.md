# SDR Collection / Acquisitions Workflow
Stephen Balogh <sgb334@nyu.edu> & Andrew Battista <ab6137@nyu.edu>

### Note

Many of the examples in this document make use of a Ruby gem called **SdrFriend**, so named because of how friendly it is. :smiley:
The codebase for that tool, as well as documentation (including on how to set it up), can be found at its [Github page](https://github.com/sgbalogh/sdrfriend).

## 1. Assessment

The collection and acquisition process begins by assessing the scope of a batch of data that you seek to acquire. The easiest way to do this is to start a spreadsheet and track out the number of layers that exist within a given collection.

#### Example Scenario
We acquire a hard disk of Shapefile (vector) layers from EastView. There are some 300 layers, grouped arbitrarily into a directory structure. First we want to get an idea of how many discrete layers (or individual Shapefiles) there are. Maybe this information is already contained in a spreadsheet provided by the vendor; if not, we may have to generate a collection of all shapefiles

```bash
cd east_view_collection
# this assumes that there's a folder called east_view_collection that has a bunch of shapefiles in it
find . -name "*.shp"
# this recursively locates all files matching the given pattern
find . -name "*.shp" > ~/Desktop/east_view_files.txt
# this saves the result to a text file, which makes it easy to see how many shapefiles exist
```
The above process is just one way to determine how many layers exist in a collection. You could also count these manually, or view the list of files in Finder hit ``command+c`` and then paste the result into a spreadsheet

## 2. Creating Repository Records / "Minting Handles"

Once the number of items in a given collection is determined, the next step is to "mint" the requisite number of IDs from the FDA. SDR assets span two different collections on the FDA, both located within the [Division of Libraries]("https://archive.nyu.edu/handle/2451/14821") community. They are:
 - [Spatial Data Repository]("https://archive.nyu.edu/handle/2451/33902") `dspace id: 651`
  -- By default, all items in this collection are public and world-accessible; accordingly, only open-data or non-licensed datasets should go here
 - [Spatial Data Repository (Private)]("https://archive.nyu.edu/handle/2451/33903") `dspace id: 652`
  -- By default, all items in this collection are visible only to logged in FDA users. The FDA is configured to use NYU SSO, so in effect this means that all items in this collection are visible only to users with NYU login credentials

You should determine which of the two collections you want to create records in. Typically, in the course of accessioning a single collection, you will interact only with one of the two.

**SdrFriend** can be used to mint, or pre-allocate, FDA records in the relevant collection. In particular, the `fda:mint` rake task allows you to pre-allocate an arbitrary amount of records and save a receipt to a text file

```bash
cd ~/git/SdrFriend
# The command bundle exec rake -T # allows you to view all available tasks
bundle exec rake fda:mint[private,100,"/Users/sgb334/Desktop/new_handles.csv"]
# First part: Is the collection public or private? (in this case private)
# Second part: How many IDs do you want? (in this case 100)
# Third part: The path to where you want the new file to be saved and your name for it (in this case new_handles.csv)
```
If the above command is successful and no errors were thrown, you should have saved a CSV text file in the format  "*handle*,*dspace-internal-id*". Hold on to this! You will need it for the next step.

*Additional note: for each of these commands, spacing and capitalization matters, so it's easiest to copy them directly from this documentation.*

### 2b. Updating the Handle table

The FDA (confusingly) uses two types of IDs to refer to records: an internal id (referred to as "dspace_id" in this document), and a Handle identifier. If only these were the same!

The Handles are great for public consumption, but unfortunately the REST API for the Faculty Digital Archive (i.e., the way in which programmed tools interact with the repository) require the internal dspace_id, and there's no simple way to turn one into another. As a work around, we manage our own lookup table, stored in the [edu.nyu repository]("https://github.com/OpenGeoMetadata/edu.nyu/blob/master/misc/handle-dspace-lookup.csv") on GitHub.

**Make sure to update this table whenever you pre-allocate / "mint" new FDA records.**

One way to make updates to the lookup table is to set up git to commit to the edu.nyu metadata repository (see [these instructions on how to do it](https://help.github.com/articles/set-up-git/)). If you're already set up to commit to the edu.nyu repository, here's a simple way to do that:

```bash
cd ~/git/edu.nyu
git pull
git checkout master

## Now we can append the CSV file of new Handles to the end of the lookup table.
## To do this, the easiest way is copy the direct output of the handles and DSpace IDs as it appears in Terminal.
## Then, go to the lookup table folder within the misc folder in the edu.nyu repository on your drive, open the .CSV with Atom, and paste the new handles in there. Make sure there are no extra lines at the end.
## Then, run this next command

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
# this stipulates the path where you want to save the file and the name of the file
```
The first data you should add is the dspace_id and Handle attributes from the collection you pre-allocated. Go ahead and paste them into the CSV. Feel free to add in additional columns to the CSV document if it's convenient to do so, though **only the columns listed in the template** will persist after converting the CSV to actual GeoBlacklight records.

Also, very important: **don't rename any of the existing template columns.**

From here, do whatever you need to do to fill in the rest of the columns. In general, you are responsible for all of the metadata elements; the only ones which are "automatically generated" for you are the `ref:download-url` and `ref:documentation-url` fields, which are intended to point to FDA bitstreams. That workflow is described below. If you are using the Google Sheets template, the `layer_modified_dt` field should also automatically update as you work in the Sheet.

Every FDA/GeoBlacklight layer needs at least one download url (the main, `ref:download-url` value). If your layer comes with additional documentation or codebook material, you may also want to link directly to that in the GeoBlacklight record (via `ref:documentation-download`), and make use of GeoBlacklight's contextual rendering for it. If you have any questions about the rules or best practices for filling out metadata fields, refer to the [GeoBlacklight Schema](https://github.com/geoblacklight/geoblacklight/blob/master/schema/geoblacklight-schema.md).

Unfortunately, it isn't possible to predict what the bitstream URLs will be until you have already uploaded them to your FDA container records. For our purposes, we will move on once we have enough metadata to otherwise constitute minimal viable GeoBlacklight records. Later we will insert the correct download URLs.

## 4. Create bitstreams packages

At this point, we may want to actually upload our content to the digital repository (FDA). We've already pre-allocated as many records as we'll need, though currently there are no bitstreams (or metadata) in them.

First, we need to assemble the bitstreams that we will be uploading. All SDR submissions should be in a directory structure that looks like this:

```
- nyu_2451_12345/
  - DISTRICT91/
    - DISTRICT91.shp
    - DISTRICT91.prj
    - DISTRICT91.shx
    - DISTRICT91.dbf
```
*Above: a sample directory structure for a SDR Shapefile submission*

The top-level directory should be the underscore delimited Handle id, prepended with `nyu_`. Within that first directory, you can place either an additional directory with the original (vendor supplied) name and contents for a single layer, or immediately supply the single layer contents.

This directory will then be zipped up, and the resulting archive will be what we upload to the FDA.

```bash
cd ~/Desktop/shapefile_staging

## Assume I have my `nyu_2451_12345` directory here
zip -r nyu_2451_12345.zip ./nyu_2451_12345/ -x "*.DS_Store"

## Now you should have a `nyu_2451_12345.zip` file in the same directory
ls -lh *.zip
# this command is to get info on all .zip files
```

**Primary download bitstreams need to be in the format: `nyu_XXXX_XXXX.YYY` (e.g. `nyu_2451_12345.zip`).** This is the convention, and only files matching this pattern will be auto-detected by SDR-related scripts / SdrFriend.

Assembling your (optional) documentation or codebook bitstream is very similar. You will want to create a directory with the same name as the corresponding primary bitstream, but in the pattern: `nyu_XXXX_XXXXX_doc.YYY`.

## 5. Upload bitstreams to FDA

Now we need to actually upload the various primary data & codebook bitstreams (when applicable) to their respective FDA records.

We can do this either one bitstream at a time, or in batch:

```bash
bundle exec rake fda:addbit[/Users/sgb334/Desktop/nyu_2451_12345.zip]
```

The above will upload the referenced bitstream to the appropriate FDA record (i.e., the record at http://hdl.handle.net/2451/12345). This will only work, however, if the bitstream follows the naming convention, and if the Handle lookup table has an entry corresponding to this Handle. Otherwise, we would need to specify exactly where we want to upload the bitstream to, by referencing that record's internal dspace_id:

```bash
bundle exec rake fda:addbit[/Users/sgb334/Desktop/my_file.zip,53423]
```

Alternatively, if we are uploading a large number of files that are all stored in the same directory, we can use SdrFriend to batch upload all of them. For this to work, we **must** have files that follow the naming convention detailed in section 5, and which are named with Handles that appear on the lookup table.

```bash
bundle exec rake fda:batch_addbit[/Users/sgb334/Desktop/uploads]
# All of the files to upload must be in the specified folder
```

## 6. Upload basic descriptive metadata to FDA and retrieve bitstream download URLs to plug back into GeoBlacklight metadata
Once all primary data bitstreams and documentation bitstreams have been uploaded to the FDA, we also need to push our agreed upon rudimentary descriptive metadata to the FDA as well. In order to do this, we will use the **SDRFriend** GeoBlacklight-to-FDA metadata command
```bash
bundle exec rake fda:gbl_to_fda_metadata[/Users/andrewbattista/git/edu.nyu]

## Here, repository path is the folder where the newly created GeoBlacklight files reside. This can be the canonical OpenGeoMetadata repository or it can be a subset you create.
```
The second part of this step is using **SDRFriend** to retrieve the existing bitstream download URLs, which are important elements for GeoBlacklight metadata. In order to do this, run the following command:
```bash
bundle exec rake metadata:bithydrate[/Users/andrewbattista/git/edu.nyu]
# As above, this can be the canonical repository in OpenGeoMetadata or it can be a subset that you create.
```
Once you have discovered the metadata for the FDA record(s) in question, extract the bitstream URLs and stick them back into the CSV template that you have been working on.... (instructions TBA)

## 7. Finalize GeoBlacklight Metadata Records and Export as .JSON
Now that you have a completely filled out CSV, use **SDRFriend** to transform it into a single JSON file. First, download or save the file you're working on as a CSV. Then, run the following command:
```bash
bundle exec rake metadata:csv_to_json[/Users/sgb334/Downloads/eastview_files.csv,eastview_files.json]
# First part: Path and location where your CSV is
# Second part: Name of your new .JSON file
```
Doing this will generate a single file with as many .JSON records as there are rows in your CSV. The next step is to use **SDRFriend** to split this one individual .JSON record into individual item records that are named `geoblacklight.json` and reside in folders named according to our handle naming structure convention. In order to do this, run the following command:
```bash
bundle exec rake metadata:split[eastview_files.json,/Users/sgb334/git]
# First part: Name of single .JSON file
# Second part: Destination path where you want the newly created files to reside.
# For the sake of ease, it's recommended to put these into the same folder where git is configured on your computer
```
Once these folders and files are created, you're ready to move on to step 10, committing these records to the OpenGeoMetadata repository (below).
## 8. Create SQL versions of datasets and upload to PostGIS
*Note: This step is only relevant for vector files. If you are accessioning raster images, such as GeoTIFFs, skip to step 8.*

To enable previews and downloads via GeoServer's WMS / WFS services, we have to create a SQL version of our Shapefiles (in the projection EPSG:4326), and add them to the PostGIS database which stores EPSG:4326"preview" versions of every vector layer in our collection.

Earlier, while assembling bitstream packages for upload to the FDA, the procedure was to simply package up an original geospatial layer. Now, however, we'll have to make some modifications to it, namely:

- Reproject the layer to the standardized CRS of `EPSG:4326` (if it isn't already in that CRS)
- Rename the file to represent the Handle associated with it
  -Earlier we were just naming the containing folder using the Handle convention, and letting the Shapefile keep its original name; now we have to make sure the layer uses the Handle as its name, so that the resulting table on PostGIS is predictably named
- Create a SQL version of the Shapefile
  -- This involves converting a Shapefile into a SQL script that creates a new table (with name in the pattern `nyu_XXXX_XXXXX`)
- Connect to the PostGIS database on AWS and insert the new layer using its SQL script

There is only a single PostGIS database, and it is connected to by both the Public and Restricted GeoServer VMs -- the distinction between "public" and "restricted" is not represented anywhere at the PostGIS level; this is fine, because our PostGIS database is not for consumption by any user other than GeoServer.

See Additional Section `b` below for details on how to perform the conversions and update described above, using a script or some command-line tools.

## b. Upload raster images to GeoServer

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
##  Enable layers on GeoServer

GeoServer can be interacted with through a web-interface, or a REST HTTP API.

### Fundamentals

We are maintaining two GeoServers:

- Maps-Public ([maps-public.geo.nyu.edu](http://maps-public.geo.nyu.edu))
 -- Accessible to everyone in the world
 -- Contains layers for all NYU hosted records with `"dc_rights_s": "Public"
- Maps-Restricted ([maps-restricted.geo.nyu.edu](http://maps-restricted.geo.nyu.edu))
  -- Should be accessible only to NYU IP range, and to other hosts in the AWS virtual cloud
  -- Contains layers for all NYU hosted records with `"dc_rights_s": "Restricted"

Each GeoServer has access to map data in two ways:
- Vector
  -- Vector data is served through a database connection to our AWS-hosted PostGIS
- Raster
  -- Raster data is served using files stored on two EBS volumes (one for the Public host, one for the Restricted host), directly mounted on each respective GeoServer host at `/ebs`

### Via web interface


### Via API

## Commit records to repository

Once you have finalized your metadata records (see step 7), make sure to commit them to the [`edu.nyu` repository](https://github.com/OpenGeoMetadata/edu.nyu). Best practice is to commit your changes on a new collection branch, then issue a pull request from that branch into MASTER.

A Travis continuous integration (CI) script on GitHub should kick off for any commit. It will attempt to validate every record (using [GeoCombine](https://github.com/OpenGeoMetadata/GeoCombine)), and log the results.

Here's a sample workflow with Git:

```bash
cd ~/git/edu.nyu
git checkout master
git pull ## Make sure you are up to date

## Switch to a new local branch, based on master
git checkout -b my_new_collection
```

At this point, I'll add all of my new records to the `~/git/edu.nyu` directory. Once the new files are in place, I can commit and push to GitHub.
```bash
git status ## just to inspect the changes
```
```bash
git add . ## Stages all changes
git commit -m "Adds records for whatever collection"

## Push the commit to a new branch on Github
git push --set-upstream origin my_new_collection
```

Now it's off to Github. If you go to the [main page for the repository](https://github.com/OpenGeoMetadata/edu.nyu), you should be able to see that a new branch (`my_new_collection`) was just pushed. Take a look at the branch, and if everything seems good, issue a pull request.

After that pull request is approved and merged into MASTER, make sure to delete the `my_new_collection` branch on Github, since it is no longer needed, and then on your local machine:

```bash
git checkout master
git pull
```

And you're gcords on Solr

## Additional / optional steps

## a. Calculating bounding boxes for `solr_geom` field

All GeoBlacklight records require a bounding box, which is used both for display purposes (it controls the default zoom, and also is displayed on the index page), as well as indexed for use in the spatial search.

There are a lot of ways to generate this data, but you will probably want to do that automatically. GDAL is useful for this.

SdrFriend provides a wrapper over GDAL, and a utility for generating `solr_geom` syntax bounding boxes, given a Shapefile. Note that the Shapefile should be in EPSG:4326 (WGS 84) CRS.

```bash
rake gdal:bounding[/path/to/file.shp]
```

Or, alternatively, you can use a SdrFriend task for finding all Shapefiles, recursively, existing within a directory, and have the output printed out for all of them:

```bash
rake gdal:bounding_many[/path/to/shapefile_directory]
```

## b. Using the `vector-processing-script` tool

If you're reading this documentation, you should also have access to a script located in a directory called `vector-processing-script`. This provides a simple command-line interface to some common data-processing steps. It is essentially just a wrapper on top of GDAL / OGR commands.

#### Install pre-requisite packages
First, you need to make sure that you've installed all of the prerequisites. On a Mac that has HomeBrew installed, that might look like this:

```bash
brew update
brew doctor

brew install postgresql ## psql
brew install gdal ## ogr2ogr
brew install postgis ## shp2pgsql
```

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


## c.  Caching tile layers to allow for easier page loading

## d. Miscellaneous tips/tricks/documentation

- Link to FDA API primer

- [Removing .DS_Store files](https://jonbellah.com/articles/recursively-remove-ds-stort)
