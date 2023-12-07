# Deploy the GeoBlacklight Rails App

## Context 

As of fall 2023, the SDR Rails app is deployed to our AWS enviroments (both staging and production) using [Capistrano](https://capistranorb.com/).

## Prerequisites
- Git
- Ruby (via RVM, Chruby, or Rbenv)


## Instructions
1. Ask DLTS DevOps for the pem key and then move it into the `~/.ssh` directory on your local machine. (Don't rename the file when you move it!)
2. Clone [the application repo](https://github.com/NYULibraries/spatial_data_repository) & cd into it
   ``` sh
   gh repo clone NYULibraries/spatial_data_repository && cd spatial_data_repository
   ```
4. Install the ruby dependencies
   ```sh
   bundle install
   ```
6. Deploy the app to the [staging instance](https://geo-stage.library.nyu.edu/). The command will deploy the codebase and also restart the Apache server for you, so changes should be live right away.
   ```sh
   bundle exec cap staging deploy
   ```
8. After QA testing the app on staging, deploy the application to [production](https://geo.nyu.edu):
    ``` sh
    bundle exec cap production deploy
    ```

> NOTE: Your locally cloned codebase is NOT what's being deployed when you run `cap deploy`! You are are actually just using your local permissions and the config settigs in the local repo to deploy the contents of the `main` branch of [NYULibraries/spatial_data_repository repository](https://github.com/NYULibraries/spatial_data_repository) on GitHub. So make sure what's in `main` on GitHub is what you're ready to deploy.
