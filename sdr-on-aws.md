# SDR on AWS Architecture
Stephen Balogh <sgb334@nyu.edu>, 5/10/2018

## Overview

The Spatial Data Repository (SDR) relies on virtual hosts, object storage, and databases hosted on Amazon Web Services.

As a service, it consists of:
- A GeoBlacklight (Ruby on Rails) based catalog
- A Solr index used by the GeoBlacklight catalog, storing metadata for all records
- Two instances of GeoServer, a web application that turns spatial data into preview tiles and makes data accessible through WMS and WFS mapping standards
- A PostgreSQL (with PostGIS extensions) database that stores vector datasets used by GeoServer
  - Only vector data is stored on PostGIS; raster imagery is storaged as files on an EFS share that is mounted by the two GeoServer hosts
- A MySQL database used by GeoBlacklight to store user information, and anonymized query logs
- The Faculty Digital Archive, which provides access to bitstreams of original datasets

## Elastic IPs

| IP Address        | Domain           | EC2 Host  | Note |
| ------------- |:-------------| -----:| ------ |
| `52.4.204.0` | [geo.nyu.edu](https://geo.nyu.edu) |  `P1_1:blacklight-rails` | |
| `52.3.70.94` |  [maps-public.geo.nyu.edu](https://maps-public.geo.nyu.edu) | `P1_1:geoserver-public` | |
| `52.2.192.218` | [maps-restricted.geo.nyu.edu](https://maps-restricted.geo.nyu.edu) | `P1_1:geoserver-restricted` | |
| `52.21.211.78` | [metadata.geo.nyu.edu](https://metadata.geo.nyu.edu) | `P1_1:metadata` | |
| `54.174.220.44` | -- *no domain* -- | `P1_1:solr-core` | This host should never be accessible to hosts outside of the virtual private cloud, so it needs no domain. |
| `52.200.118.126` | -- *no domain* -- | ` ` | Typically used for staging Blacklight |
| `52.70.140.149` | -- *no domain* -- | ` ` | |
| `52.70.140.26` | -- *no domain* -- | ` ` | |

## Host Descriptions

### EC2: `P1_1:blacklight-rails`

This hosts the main GeoBlacklight application. Apache with Phusion Passenger is the web server responsible for hosting it (as a Ruby on Rails application).

#### Configuration / Details
- Apache needs the Phusion Passenger module enabled.
- The host uses Apache2’s ability to enable/disable different site configurations (which are stored in `/etc/apache2/sites-available`). The production site configuration is called `nyu_geoblacklight_production`, which includes a directive for Phusion Passenger to load the Rails application in the “production” environment.
- The Rails application resides in `/home/ubuntu/nyu_geoblacklight`
- Host makes use of an EBS volume that should always be mounted at `/blacklight_data`; this stores cached downloads from GeoServer, and may periodically need to be flushed (files can simply be deleted from here)

#### SSL
- Certificates are stored at `/ssl/geo.nyu.edu`, and used in the Apache2 configuration

#### Connections
- Read access to `geoblacklight-db` (RDS) MySQL database
- Connection to `P1_1:solr-core` on port `8983`
- Connection to `P1_1:maps-restricted` on `80`, `443`
- World-accessible on `80` and `443` (Apache will force redirect from `80` to `443`)
- SSH (`22`) accessible only as needed

##### Links
- Documentation on [installing Phusion with Apache](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-rails-app-with-passenger-and-apache-on-ubuntu-14-04)


### EC2: `P1_1:geoserver-public`

This is one of the two GeoServer hosts. This particular one is responsible for web services that provide access to any “public” datasets in the NYU SDR collection. It should be accessible to all IPs. GeoServer is hosted with tomcat7, which is managed as a service.

#### Configuration / Details

- Expects an EBS volume mounted at `/ebs/geoserver_public`
  - This contains the GeoServer home, which includes records of all loaded layers, and a large tile cache
- Expects an EFS share mounted at `/efs/geoserver`
 - This contains all raster layers (public and restricted), and is the same EFS share that is mounted by the maps-restricted GeoServer; only the public layers will be loaded on this GeoServer host’s configuration, however
- Java heap (and other variables) can be configured in `/usr/share/tomcat7/bin/setenv.sh`
- Tomcat7 conf dir is at `/var/lib/tomcat7/conf`

#### SSL
- SSL certificates are at `/ssl/maps-public.geo.nyu.edu`
  - Tomcat uses a **keystore** for SSL certificates; this is configured in `/var/lib/tomcat7/conf/server.xml`

#### Connections
- World accessible on `80` and `443`
- SSH (`22`) access as needed

##### Links
- Documentation on [configuring SSL with GeoServer/Tomcat](http://docs.geoserver.org/stable/en/user/security/tutorials/cert/index.html)


### EC2: `P1_1:geoserver-restricted`

This is the second of the two GeoServer hosts. It is responsible for hosting all “restricted” datasets in the NYU SDR collection. It should be accessible only to NYU IP address ranges, and to P1_1:blacklight-rails. GeoServer is hosted with tomcat7, which is managed as a service.

*The configuration details are almost identical to that above, for the public GeoServer: `P1_1:geoserver-public`.*

#### Configuration / Details
- Expects an EBS volume mounted at `/ebs/geoserver_restricted`
- Expects an EFS share mounted at `/efs/geoserver`

#### SSL
- SSL certificates are at `/ssl/maps-restricted.geo.nyu.edu`

#### Connections
- NYU IP range accessible on `80` and `443`
  - GeoServer is not authentication-aware in any way; rather, it is firewalled to only be accessible to NYU IPs, and then we use OCLC EZproxy to provide off-campus access (similar to any licensed database)
- Directly accessible to `P1_1:blacklight-rails`, and `P1_1:metadata` on `80` and `443`


### EC2: `P1_1:metadata`

This is something of an "odds and ends" host. It is useful as a gateway into the private cloud, since it has the ability to directly connect to the Solr instance, our two RDS databases, and all other hosts. We use it to upload tables to PostGIS (PostgreSQL), perform database backups, index records to the production Solr core, and other miscellaneous tasks. It also hosts an Omeka instance used (occasionally) for metadata creation.

#### Configuration / Details
- Vector shapefile conversion area is at `/home/ubuntu/vector-processing`
- Metadata records and scripts for indexing to Solr are at `/home/ubuntu/metadata`
  - The directory `/home/ubuntu/metadata/production` contains records that should be live in our production Solr core
  - An indexing script is at `/home/ubuntu/metadata/index-records.rb`

#### SSL
- SSL certificates are at `/ssl/metadata.geo.nyu.edu`

#### Connections
- Should be world-accessible on `80` and `443`, only for purposes of Omeka access
- SSH (`22`) access to NYU workstations as needed
- Connection to `P1_1:solr-core` on port `8983`
- MySQL access
- PostgreSQL access

### EC2: `P1_1:solr-core`

#### Configuration / Details
  - Solr has no authentication mechanism, so it **must be firewalled** at all times, and only be accessible to Blacklight (`P1_1:blacklight-rails`), and to the host responsible for indexing (`P1_1:metadata`)
  - The production Solr core is `blacklight_core`
    - Therefore, the endpoint of the relevant production core would be: `http://54.174.220.44:8983/solr/blacklight_core`
  - Version of Solr is 5.5, but later versions should work without a problem too
  - Solr cores are stored on an EBS volume attached at `/solr_home`
  - Solr is run as a service (e.g. you can `sudo solr service status`)
  - Environment for Solr can be configured in the startup script located at `/etc/default/solr.in.sh`
    - Main customizations to note are:
      - `SOLR_HOME=/solr_home/data`
      - `SOLR_HEAP="2048m"` (or higher)

#### SSL
  - **No SSL certificates**; this host will only be communicated with from within the virtual private cloud

#### Connections
  - Accessible to `P1_1:blacklight-rails` on `8983`
  - Accessible to `P1_1:metadata` on `8983` and `22`

## RDS Databases

| Type | Endpoint | Port | Database Name | Note |
| ---- |:-------- | ---- | ------------- | ---- |
| PostgreSQL + PostGIS | `nyu-geospatial.cfh3iwfzn4xy.us-east-1.rds.amazonaws.com` | 5432 | `geospatial_data` | Vector data stored here|
| MySQL | `geoblacklight-db.cfh3iwfzn4xy.us-east-1.rds.amazonaws.com` | 3306 | `geoblacklight_prod_db` | Production Blacklight user db |
| MySQL | `geoblacklight-db.cfh3iwfzn4xy.us-east-1.rds.amazonaws.com` | 3306 | `geoblacklight_dev_db` | Development Blacklight user db |
