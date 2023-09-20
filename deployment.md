# Deploying SDR Services (using `capistrano`)

Using `capistrano` to deploy SDR is a simple process you can perform from your local environment. 
You will need the following:

- A local clone of the SDR repository
- the SSH key for the staging/production servers (See DevOps for this key)
- connection to to the NYU VPN (if off-campus)


## Deployment Commands

Deploying to `staging` is this command:

`cap staging deploy`

Deploying to `production` is this command:

`cap production deploy`
