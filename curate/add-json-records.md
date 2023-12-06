# Add and validate GeoJSON records in GitHub

## Context
We have two repositories where we store metadata records for the SDR:
- [opengeometadata/edu.nyu](https://github.com/opengeometadata/edu.nyu) is where cannonical *production* records live; we and other institutions will index records from these repos directly to their GeoBlacklight instances.
- [nyu-dataservices/gis-metadata-staging](https://github.com/gis-metadata-staging) is a *fork* of opengeometadata/edu.nyu; we use it to work on records that are in process and to index to our NYU SDR staging instance of GeoBlacklight.

All new NYU records should be made in [nyu-dataservices/gis-metadata-staging](https://github.com/gis-metadata-staging), validated, then merged via PR over to [opengeometadata/edu.nyu](https://github.com/opengeometadata/edu.nyu) when ready. Both repositories should be in-sync whenever possible.


TO DO — ADD DIAGRAM OF INSTRUCTIONS HERE 
 
## Instructions

1. You can add records or record fixes to [NYU-DataServices/gis-metadata-staging](https://github.com/NYU-DataServices/gis-metadata-staging) in a new branch (e.g., `batch-lidar-2023` or `patch-dct-language-sm`) from your local machine or directly in the GitHub GUI. Just make sure the records are in the correct place for their schema, e.g., `metadata-1.0` or `metadata-aardvark`. *(Note: Only aardvark records will get indexed to the NYU Spatial Data Repository as of Fall 2023.)*
2. When the records in the branch are ready, make a pull request from your branch to `main` in [the same staging repo]((https://github.com/NYU-DataServices/gis-metadata-staging)). This will trigger linting for the records in GitHub Actions. (See badges above)
3. *** *TO DO * ** After the linters & PR pass, merging into `main` will trigger indexing of the aardvark records found in the `main` branch of [NYU-DataServices/gis-metadata-staging](https://github.com/NYU-DataServices/gis-metadata-staging) to NYU's [staging instance of the SDR](https://geo-stage.library.nyu.edu/) for QA testing.* *(Must be on NYU VPN to view the staging instance)*
4. After QA on staging, a PR can be submitted from [NYU-DataServices/gis-metadata-staging](https://github.com/NYU-DataServices/gis-metadata-staging) over to the cannoinical repo [OpenGeoMetadata/edu.nyu](https://github.com/OpenGeoMetadata/edu.nyu). Any records pulled into this cannonical OpenGeoMetadata repo will get indexed to our production instance (and other institutions'!)

## Local development & validation

Follow the steps below to run linting and other tasks locally.

### Prerequisites 
- Git
- Ruby (via RVM, Chruby, or Rbenv)
  
### Steps
1. Clone the repository
    ```
    gh repo clone NYU-DataServices/gis-metadata-staging & cd gis-metadata-staging
    ```
2. Install ruby dependencies
    ```
    bundle install
    ```
3. Run linting on one or both groups of records
   ```
   bundle exec rake lint:v1
   bundle exec rake lint:aardvark
   bundle exec rake lint:all
   ```
4. If the aarvark linters fail, you might try to run and/or update the script in `/misc`:
   ```
   bundle exec ruby misc/aardvark-schema-patching.rb
   ```
