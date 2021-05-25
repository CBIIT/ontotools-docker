# OntoTools Deployment Pipeline

This repository contains the official Dockerised deployment pipeline for the EBI OntoTools stack:

* The Ontology Lookup Service (OLS)
* The Ontology Xref Service (OxO)
* ZOOMA

# Instructions

First, install Git, Docker and Docker compose. On an Ubuntu server:

    apt install git docker.io
    sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

To use Docker without `sudo`, make sure your user is in the `docker` group. For example, if your username is `spot`:

    sudo usermod -aG spot docker
   
Next, clone this repository:

    git clone https://github.com/EBISPOT/ontotools-docker-config.git
    cd ontotools-docker-config
   
Finally, run the `redeploy.sh` script to deploy the OntoTools stack:

    ./redeploy.sh

To change customisation options, edit the `docker-compose.yml` file and re-deploy. For example, to change the title of your OLS instance, edit the `ols.customisation.title` line.

# Pipeline

The OLS/OXO/Zooma pipeline (just "pipeline" from now on) supports the following workflows:

1. Deploying a new OLS/OXO/Zooma instance entirely using docker containers, using `docker-compose`.
2. Re-indexing OLS/OXO when the data changes.

The pipeline performs the following steps, which are encoded as as series of docker commands in [redeploy.sh](redeploy.sh). Note that [update-data.sh](update-data.sh) can be used to _just_ reindex the data, without actually stopping most of the services. It is, due to the low overall runtime of the script, not necessary to use update-data.sh (you can simply always use redeploy.sh).

1. Starting ols-solr and ols-mongo instance
2. Import OLS config from [config/ols-config](config/ols-config).
3. Index ontologies in OLS
4. Start the remaining services (ols-web oxo-solr oxo-neo4j oxo-web). It is important that ols-web is not running at indexing time. This is a shortcoming in the OLS architecture and will likely be solved in future versions.
5. Extract all datasets from OLS for OxO processing
6. Load datasets (not mappings) into OxO Neo4j
7. Extract the xref mappings from OLS and exports them into OxO format.
8. Loads the mappings into OxO Neo4j
9. Index mappings in solr

## Custom installations

We have started to maintain a [list of known custom OLS installations](docs/custom_ontotools_users.md). Please create an issue if you want your installation to be listed as well.
