# SDR Documentation

This repository is *the* place for documentation relevant to NYU's [Spatial Data Repository](https://geo.nyu.edu) and the parts of NYU Libraries infrastructure used to maintain it.

> **As of Fall 2023, this repo is being pruned and reorganized as part of a major SDR upgrade sprint; old documentation will be available in the `archive` branch, and only docs vetted as currently correct and useful will make their way to `main`. (WiP)**

## SDR Quicklinks

### Deployments
- Production: [geo.nyu.edu](https://geo.nyu.edu)
- Staging: [geo-stage.library.nyu.edu](https://geo-stage.library.nyu.edu/) (requires nyu vpn)

### Repositories
- Application: [nyulibraries/spatial_data_repository](https://github.com/nyulibraries/spatial_data_repository)
- Documentation: [nyulibraries/sdr-documentation](https://github.com/nyulibraries/sdr-documentation)
- CLI Tool: [nyulibraries/sdr-cli](https://github.com/nyulibraries/sdr-cli)
- Core GBL: [geoblacklight/geoblacklight](https://github.com/geoblacklight/geoblacklight)
- NYU GIS Records (Prod): [opengeometadata/edu.nyu](https://github.com/opengeometadata/edu.nyu)
- NYU GIS Records Staging: [nyu-dataservices/gis-metadata-staging](https://github.com/nyu-dataservices/gis-metadata-staging)

### Misc
- [2023 Documentation Scan](https://docs.google.com/spreadsheets/d/1cEoenvQXXhaA6mCGs4gtJlEN_AasXOArnFJa-561Kq0)
- [Google Drive](https://drive.google.com/drive/folders/0B1StS1CI0jNRfmRkZ1BHbXRUdkJlemFoeUNsRDdxOUxUdVotTWpqdmJfRFFSei1DTUc0cUE)

### Deprecated/Archived
- [nyulibraries/sdrfriend](https://github.com/nyulibraries/sdrfriend)
- [nyu-dataservices/gis-metadata-sandbox](https://github.com/nyu-dataservices/gis-metadata-sandbox)
- [spatial_data_repository wiki](https://github.com/NYULibraries/spatial_data_repository/wiki)


## Contents

### Maintain & Manage
- AWS Architecture
- [Deploy the GeoBlacklight Rails App](maintain/deploy.md)
- Secrets Management
- Roadmap

### Develop
- Developing Locally (Mac)
- Contributor guide
- App changelog

### Curate & Deposit
- Accessioning Workflows
- [Add and validate GeoJSON records in GitHub (staging and cannonical OpenGeoMetadata repos)](curate/add-json-records.md)
- Index Records to Staging and Prod Solr
- Use FDA API
- Update Featured Collections and Maps

### Test
- QA guide
- Running CI tests
- [Validate GeoJSON Records to the JSON Schema(s)](curate/add-json-records.md#local-development--validation)


