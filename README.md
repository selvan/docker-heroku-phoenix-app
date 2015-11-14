# Deploying phoenix app to heroku

## Pre-requisites

- Docker : http://docs.docker.com/v1.8/installation/
- Heroku toolbelt : https://toolbelt.heroku.com/

## Install heroku docker plugin
    heroku plugins:install heroku-docker
    
## Copy app.json and Procfile
    Copy app.json and Procfile from this repository to your phoenix application root folder 

## Create docker related files (Dockerfile and docker-compose.yml) 
    heroku docker:init

## Append following lines to Dockerfile
    # Compile elixir files for production
    ENV MIX_ENV prod
    
    # This prevents us from installing devDependencies
    ENV NODE_ENV production
    
    # This causes brunch to build minified and hashed assets
    ENV BRUNCH_ENV production
    
    # We add manifests first, to cache deps on successive rebuilds
    COPY ["mix.exs", "mix.lock", "/app/user/"]
    RUN mix deps.get
    
    # Again, we're caching node_modules if you don't change package.json
    ADD package.json /app/user/
    RUN npm install
    
    # Add the rest of your app, and compile for production
    ADD . /app/user/
    RUN mix compile \
        && brunch build \
        && mix phoenix.digest


## Edit config/[dev|test|prod.secret].exs
	# Configure your database
	config :chatapp, Chatapp.Repo,
	  adapter: Ecto.Adapters.Postgres,
	  url: {:system, "DATABASE_URL"},
	  pool_size: 20

## Create a heroku app 
    heroku create

## Push docker image to heroku
    heroku docker:release
    
- Executing 'heroku docker:release' command for very first time will take some time, subsequent runs will be much quicker.

## Creating database
    No need to run "mix ecto.create" with heroku as we just use postgresql database in heroku, rather than creating it.

## Running Ecto migrations
    heroku run "cd /app/user;MIX_ENV=prod NODE_ENV=production BRUNCH_ENV=production mix ecto.migrate"

## Open application URL in browser
    heroku open

## Miscellaneous heroku commands
### Connect to postgresql cli
    heroku pg:psql DATABASE
    
- Note: Run above command as it is, without replacing DATABASE with anything 

### Reset database
    heroku pg:reset DATABASE_URL
    
- Note: Run above command as it is, without replacing DATABASE_URL with anything

### Heroku logs
    heroku logs -t --app salty-mesa-7188
