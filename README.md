eks-od-datastore-pusher
=======================

This scripts takes individual CSV files harvested from portal.eks.sk (EKS)
(with [eks-od-harvestrer](https://github.com/OpenDataSk/eks-od-harvestrer))
and uploads them into single resource in CKAN. Intended side-effects of that
are:

- merging of the individual files into one big table
- calculation of "last modified" timestamps, since EKS is not providing
  those

# Basic information

This is a second stage of "liberating data" from EKS. (First stage is
[eks-od-harvestrer](https://github.com/OpenDataSk/eks-od-harvestrer)). This
second stage combined together multiple CSV files into one table into
[https://odn.opendata.sk/dataset?tags=eks] (utilizing CKAN's DataPusher) to
allow easier data re-use (some API calls, etc.).

## Usage

* Clone this repository:

        git clone https://github.com/OpenDataSk/eks-od-datastore-pusher.git
        cd eks-od-datastore-pusher

* Create a virtualenv and install dependencies:

        virtualenv .env
        source .env/bin/activate
        pip install -r requirements.txt

* Define your CKAN URL and API key in the `config.ini` file:

	cp config.ini.template config.ini
	...

* Run the setup command, and write the resulting resource id in your `config.ini` file:

        python datastore_update.py setup

* Run the update command:

        python datastore_update.py update

You probably want to set up this command to run hourly, eg with a cron job:

    crontab -e

Add a line like this:

    0 0 * * * /path/to/your/pyenv/bin/python /path/to/your/workspace/eks-od-datastore-pusher/datastore_updater.py update

## License

This code is BSD licensed, see [the license](LICENSE).

## Contribution

Simply send a pull request or report an issue. Thank you in advance.

## TODO

- add license (for now BSD preffered)
- detect duplicates (those do occur in EKS data, we can't put them into DataStore, we should initially at least report that)
- use loggers (instead of print), set log level via config.ini
- ...

# Detailed technical stuff

## How it works

When running the `setup` command we are doing the following things:

* Creating a new dataset in the remote CKAN instance using the [package_create](http://docs.ckan.org/en/latest/api/index.html#ckan.logic.action.create.package_create) API action.

* Preparing a mapping of the table fields with the correct field types to ensure they are handled correctly by the DataStore.

Once we have this initial setup we can use the `update` command to periodically
walk through CSV files (harvested by `eks-od-harvestrer`) and push contents
of each into DataStore table using the [datastore_upsert](http://docs.ckan.org/en/latest/maintaining/datastore.html#ckanext.datastore.logic.action.datastore_upsert) API action.

As we defined a primary key when creating the DataStore table we can use the
`upsert` method, which will update existing records and insert any new ones.

When accessed via the CKAN frontend, the data can be explored in the grid
and map previews powered by Recline, and of course it can be accessed
programmatically from other applications using the
[datastore_search](http://docs.ckan.org/en/latest/maintaining/datastore.html#ckanext.datastore.logic.action.datastore_search)
API action.
