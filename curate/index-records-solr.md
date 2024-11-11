# Index Records to Staging and Prod Solr

## Index to Prod

1. SSH into staging instance and cd into current deployment:
    ``` sh
    cd ~/nyu_geoblacklight/current
    ```
2. Pull the up-to-date records from OpenGeoMetadata repositories. You can update all records or only NYU's records by running either:
    ```
    bundle exec sdr-cli pull
    ``` 
    or: 
    ```
    bundle exec sdr-cli pull --repo edu.nyu
    ```
    respectively.
3. Index all JSON records in the `tmp` directory to the Solr at `SOLR_PROD_URL` (look that URL up [here](https://docs.google.com/spreadsheets/d/1jBfnJ40QW57Ysh0OGn8tB2ya-aa4mhexwju22-Q3TK8/edit#gid=0)); NOTE: Make sure it starts with `http://`, ends with `:8983/solr/blacklight-core/` and contains the SOLR URL, not the IP!
    ```
    bundle exec sdr-cli index --directory='./tmp/opengeometadata' --solr_url='http://{SOLR_PROD_URL}:8983/solr/blacklight-core/'
    ```

## Index to Staging

1. SSH into staging instance and cd into current deployment:
    ``` sh
    cd ~/nyu_geoblacklight/current
    ```
2. Clear out previous nyu.edu records stored in the local folder if already cloned:
    ``` sh
    mkdir -p tmp/opengeometadata && rm -rf tmp/opengeometadata/edu.nyu/
    ```
3. OPTIONAL!!: Clear out live Solr index to get a clean slate (see: https://stackoverflow.com/questions/23228727/deleting-solr-documents-from-solr-admin).. This would ensure the Solr core only includes records you'll index in the next step. But it is not a requirement.
4. Clone records from our staging repository into the directory (`tmp/opengeometadata/edu.nyu`) that `sdr-cli` expects NYU records to be in:
    ``` sh
    git clone https://github.com/NYU-DataServices/gis-metadata-staging tmp/opengeometadata/edu.nyu
    ```
5. OPTIONAL!!!: Clone other opengeometadata repos if you want to update/index those records too. (NOTE: It'll take forever). This is only relevant if you want to confirm that other institions records look right in our instance. It helps for testing app behavior instead of testing the structure of our NYU records.
    ``` sh
    bundle exec sdr-cli clone
    ```
7. Index all JSON records in the `tmp` directory to the solr at `SOLR_STAGING_URL`(look that URL up [here](https://docs.google.com/spreadsheets/d/1jBfnJ40QW57Ysh0OGn8tB2ya-aa4mhexwju22-Q3TK8/edit#gid=0)); NOTE: Make sure it starts with `http://`, ends with `:8983/solr/blacklight-core/` and contains the SOLR URL, not the IP!
    ```
    bundle exec sdr-cli index --directory='./tmp/opengeometadata' --solr_url='http://{SOLR_STAGING_URL}:8983/solr/blacklight-core/'
    ```