# Add and validate GeoJSON records in GitHub

## Context
We have two repositories where we store metadata records for the SDR:
- [opengeometadata/edu.nyu](https://github.com/opengeometadata/edu.nyu) is where cannonical *production* records live; we and other institutions will index records from these repos directly to their GeoBlacklight instances.
- [nyu-dataservices/gis-metadata-staging](https://github.com/gis-metadata-staging) is a *fork* of opengeometadata/edu.nyu; we use it to work on records that are in process and to index to our NYU SDR staging instance of GeoBlacklight.

All new NYU records should be made in [nyu-dataservices/gis-metadata-staging](https://github.com/gis-metadata-staging), validated, then merged via PR over to [opengeometadata/edu.nyu](https://github.com/opengeometadata/edu.nyu) when ready. Both repositories should be in-sync whenever possible.

## Instructions

1. Clone [NYU-DataServices/gis-metadata-staging](https://github.com/NYU-DataServices/gis-metadata-staging) and `cd gis-metadata-staging`.
2. Create a new branch (`git checkout -b [[branch name]]`) using the following conventions for branch name:

    | Action type | Branch name |
    | --- | --- |
    | Edit existing records | edit-COLLECTION-SHORT-NAME |
    | Create new records | new-COLLECTION-SHORT-NAME |

    The collection short name should be a keyword from the title of the collection. Use lower case and separate words with hyphens.

    **Example:** `git checkout -b edit-egress`

3. Edit the relevant JSON records, replace existing records with the JSON modified in Airtable, or add new records to the `metadata-aardvark` folder. (Not `metadata-1.0` *Only aardvark records will get indexed to the NYU Spatial Data Repository as of Fall 2023.)*
4. When the records in the branch are ready, make a pull request from your branch to `main` in [the same staging repo]((https://github.com/NYU-DataServices/gis-metadata-staging)), using a standardized commit message:
   - `git add JSON-FILES-CHANGED`, e.g. `git add metadata-aardvark/Datasets/nyu-2451-33876.json`
   - `git commit` <-- this will open a Vim editor window where you can write the title of the commit and a description. Hit `i`to start editing, entering your commit title on the first line after the comment section, a blank line, then the commit description.
   - Use the following convention for commit titles:
     
        | Commit title keyword | Record change type |
        | --- |----------------------------------------------|
        | Field Error | Intended field value not correct             |
        | Bitstream Edit | Change to location of download URL for asset |
        | Typo Correction | Inadvertent error in field value |
        | Object Type Error | Incorrect JSON object type (e.g. array for string in key's value |
        | General Validation Error | Fix to general linting validation fail |
        | Add New Records | Add new record to system |
   
    - For a description (entered after leaving a blank line after the commit title), offer a short narrative of a 1-2 sentences about what edits are being made.
    - Close vim using `esc`, `:`, `wq`
    - Push using `git push --set-upstream origin [[local branch name]]`, e.g. `git push --set-upstream origin edit-egress`
    - Return to the  [gis-metadata-staging]((https://github.com/NYU-DataServices/gis-metadata-staging)) repo, click on "Pull Requests" on the top menu, and select "New pull request." Set it so that the pull is from your local branch into the repo's `main` branch.

4. When a PR into `main` is created, this will trigger linting for the records in GitHub Actions. (See: [gis-metadata-staging/actions](https://github.com/NYU-DataServices/gis-metadata-staging/actions/workflows/lint.yml)). When they pass, go ahead and merge into `main`
3. *** *TO DO / NOT IMPLEMENTED * ** Commits to `main` (directly or via merged PR) will trigger indexing of the aardvark records found in the `main` branch of [NYU-DataServices/gis-metadata-staging](https://github.com/NYU-DataServices/gis-metadata-staging) to NYU's [staging instance of the SDR](https://geo-stage.library.nyu.edu/) for QA testing.* *(Must be on NYU VPN to view the staging instance)*.
4. After QA on staging, a PR can be submitted from [NYU-DataServices/gis-metadata-staging](https://github.com/NYU-DataServices/gis-metadata-staging) over to the canonical repo [OpenGeoMetadata/edu.nyu](https://github.com/OpenGeoMetadata/edu.nyu). Any records pulled into this canonical OpenGeoMetadata repo will get indexed to our production instance (and other institutions!)


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
