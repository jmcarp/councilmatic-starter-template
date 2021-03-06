# Councilmatic Starter Template
This repo provides starter code and documentation for new Councilmatic instances.

## About Councilmatic

The [councilmatic family](https://www.councilmatic.org/) is a set of web apps for keeping tabs on city representatives and their legislative activity.

Councilmatic started as a Code for America project by Mjumbe Poe, who designed the earliest version of a Councilmatic site – [Councilmatic Philadelphia](http://philly.councilmatic.org/). DataMade then implemented Councilmatic in New York City (in partnership with the Participatory Politics Foundation), Chicago (in partnership with the Sunlight foundation), and Los Angeles (in partnership with the Los Angeles County Metropolitan Transportation Authority).

To simplify redeployment of Councilmatic instances, DataMade identified two necessities: (1) Open Civic Data, a data standard for describing people, organizations, events, and bills, and (2) the abstraciton of core functionality into a separate app. And so, DataMade built `django-councilmatic` – [a django app](https://github.com/datamade/django-councilmatic) with base functionality, common across all cities, which implements the OCD data standard.

With this template, you can create a Councilmatic site for your own city, built using the `django-councilmatic` app. Read on!

## Finding Data

You need data about your city in the Open Civic Data API.

How you get your data into an instance of the OCD API is up to you. What does DataMade do? We use scrapers, which run nightly to update the API by scraping data from Legistar-backed sites operated by the cities for which we built Councilmatic. Your city may be running a Legistar-backed site, and if so, you can checkout [`python-legistar-scraper`](https://github.com/opencivicdata/python-legistar-scraper) and the [`pupa`](https://github.com/opencivicdata/pupa) framework to get a head start on scraping.

If you need examples of how to customize your scraper, look at [`scrapers-us-municipal`](https://github.com/opencivicdata/scrapers-us-municipal) as well as [DataMade](https://datamade.us/), which hosts several municipal-level scrapers. You can find information about what cities and other governmental bodies are already covered on [the OCD DataMade site](http://ocd.datamade.us/jurisdictions/).

## Getting started: Create your site

NOTE: This guide focuses on setting up your app for development. It does not discuss in detail the process of finding, scraping, and importing city data or deploying your site.

### Install OS Level dependencies

* Python 3.4
* PostgreSQL 9.4 +

### Create your own councilmatic using cookiecutter
```
pip install cookiecutter
cookiecutter https://github.com/datamade/councilmatic-starter-template
```

### Make a virtualenv and install dependencies

We recommend using [virtualenv](https://virtualenv.readthedocs.io/en/latest/)
and
[virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/install.html)
for working in a virtualized development environment. [Read how to set up
virtualenv](http://docs.python-guide.org/en/latest/dev/virtualenvs/).

Once you have virtualenvwrapper set up, do the following in your bash profile:

```bash
mkvirtualenv councilmatic
cd yourcity_councilmatic
pip install -r requirements.txt
```

Afterwards, whenever you want to use this virtual environment, run:

```bash
workon yourcity_councilmatic
```

### Update deployment settings

Look for `councilmatic/settings_deployment.py.example`. Make a copy of it so that you can customize it for your city:

```
cp councilmatic/settings_deployment.py.example councilmatic/settings_deployment.py
```

This file is important! It's where you keep the parts of your Councilmatic that should not end up in version control (e.g., passwords, database connection strings, etc., which will vary based on where the app is running and which you probably won't want other people to know). Most of these variables are generic Django settings, which you can look up in the [Django docs](https://docs.djangoproject.com/en/1.10/ref/settings/); but some of these variables require customization. This table shows only the variables that you need to customize.

<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Description</th>
            <th>Optional?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>HAYSTACK_CONNECTIONS</td>
            <td>
                Dictionary of connections used by Haystack. More on
                that <a href="http://django-haystack.readthedocs.io/en/v2.5.0/tutorial.html#modify-your-settings-py">here</a>.
            </td>
            <td>No</td>
        </tr>
        <tr>
            <td>FLUSH_KEY</td>
            <td>
                Used by the `flush` view in `django-councilmatic` to clear Django's cache.
            </td>
            <td>No</td>
        </tr>
        <tr>
            <td>DISQUS_SHORTNAME</td>
            <td>
                Will cause Disqus comment threads to be attached to
                individual bill pages.
            </td>
            <td>Yes</td>
        </tr>
        <tr>
            <td>ANALYTICS_TRACKING_CODE</td>
            <td>
                Google Analytics property ID
            </td>
            <td>Yes</td>
        </tr>
        <tr>
            <td>HEADSHOT_PATH</td>
            <td>
                Absolute path to where the data loader will save
                images of members of the legislative body that are
                often available through Legistar.
            </td>
            <td>Yes</td>
        </tr>
        <tr>
            <td>EXTRA_APPS</td>
            <td>
                List of additional django apps that you want to use
                with your custom councilmatic
            </td>
            <td>Yes</td>
        </tr>
    </tbody>
</table>



### Setup your database

Before we can run the website, we need to create a database.

```bash
createdb yourcity_councilmatic
```

Then, run migrations. (Be sure that you are "working on" the correct virtual environment.)

```bash
python manage.py migrate --no-initial-data
```

Create an admin user. Set a username and password when prompted.

```bash
python manage.py createsuperuser
```


## Import data from the Open Civic Data API

The django-councilmatic app comes with a import_data management command, which populates bills, people, committees, and events, loaded from the OCD API. You can explore the nitty-gritty of this code [here](https://github.com/datamade/django-councilmatic/blob/master/councilmatic_core/management/commands/import_data.py). Note: Earlier releases of django-councilmatic (< 0.7) use `loaddata`, instead of `import_data`.

Running `import_data` will take a while, depending on volume (e.g., NYC may require around half an hour).

```bash
python manage.py import_data
```

By default, the import_data command carefully looks at the OCD API; it is a smart management command. If you already have bills loaded, it will not look at everything on the API - it will look at the most recently updated bill in your database, see when that bill was last updated on the OCD API, and then look through everything on the API that was updated after that point. If you'd like to load things that are older than what you currently have loaded, you can run the import_data management command with a `--delete` option, which removes everything from your database before loading.

The import_data command has some more nuance than the description above, for the different types of data it loads. If you have any questions, open up an issue and pester us to write better documentation.


## Running Councilmatic locally

Now, you are ready to run your unique version of Councilmatic!

``` bash
python manage.py runserver
```

Navigate to [http://localhost:8000/](http://localhost:8000/).

## Setup search

On a Councilmatic site, users can search bills according to given query parameters. To power our searches, we use [Solr](http://lucene.apache.org/solr/), an open source tool, written in Java.

**Requirements: Open JDK or Java**

On Ubuntu:

``` bash
$ sudo apt-get update
$ sudo apt-get install openjdk-7-jre-headless
```

On OS X:

1. Download latest Java from
[http://java.com/en/download/mac_download.jsp?locale=en](http://java.com/en/download/mac_download.jsp?locale=en)
2. Follow normal install procedure
3. Change system Java to use the version you just installed:

    ``` bash
    sudo mv /usr/bin/java /usr/bin/java16
    sudo ln -s /Library/Internet\ Plug-Ins/JavaAppletPlugin.plugin/Contents/Home/bin/java /usr/bin/java
    ```

**Download & setup Solr**

``` bash
wget http://mirror.sdunix.com/apache/lucene/solr/4.10.4/solr-4.10.4.tgz
tar -xvf solr-4.10.4.tgz
sudo cp -R solr-4.10.4/example /opt/solr

# Copy schema.xml for this app to solr directory
sudo cp solr_scripts/schema.xml /opt/solr/solr/collection1/conf/schema.xml
```

**Run Solr**
```bash
# Next, start the java application that runs solr
# Do this in another terminal window & keep it running
# If you see error output, somethings wrong
cd /opt/solr
sudo java -jar start.jar
```

**Index the database**
```bash
# back in the nyc-councilmatic directory:
python manage.py rebuild_index
```

**OPTIONAL: Install and configure Jetty for Solr**

Just running Solr as described above is probably OK in a development setting.
To deploy Solr in production, you'll want to use something like Jetty. Here's
how you'd do that on Ubuntu:

``` bash
sudo apt-get install jetty

# Backup stock init.d script
sudo mv /etc/init.d/jetty ~/jetty.orig

# Get init.d script suggested by Solr docs
sudo cp solr_scripts/jetty.sh /etc/init.d/jetty
sudo chown root.root /etc/init.d/jetty
sudo chmod 755 /etc/init.d/jetty

# Add Solr specific configs to /etc/default/jetty
sudo cp solr_scripts/jetty.conf /etc/default/jetty

# Change ownership of the Solr directory so Jetty can get at it
sudo chown -R jetty.jetty /opt/solr

# Start up Solr
sudo service jetty start

# Solr should now be running on port 8983
```

**Regenerate Solr schema**

While developing, if you need to make changes to the fields that are getting
indexed or how they are getting indexed, you'll need to regenerate the
schema.xml file that Solr uses to make it's magic. Here's how that works:

```
python manage.py build_solr_schema > solr_scripts/schema.xml
cp solr_scripts/schema.xml /opt/solr/solr/collection1/conf/schema.xml
```

In order for Solr to use the new schema file, you'll need to restart it.

**Using Solr for more than one Councilmatic on the same server**

If you intend to run more than one instance of Councilmatic on the same server,
you'll need to take a look at [this README](solr_scripts/README.md) to make sure you're
configuring things properly.

## Team

* Forest Gregg, DataMade - Open Civic Data (OCD) and Legistar scraping
* Cathy Deng, DataMade - data models, front end
* Derek Eder, DataMade - front end
* Eric van Zanten, DataMade - search and dev ops
* Regina Compton, DataMade - developer

## Errors/Bugs

If something is not behaving intuitively, it is a bug, and should be reported.
Report it [here](https://github.com/datamade/councilmatic-starter-template/issues).

## Note on Patches/Pull Requests

We welcome your ideas and feedback! Here's how to make a contribution:

* Fork the project.
* Make your feature addition or bug fix.
* Commit - please be descriptive, but concise.
* Send a pull request. Bonus points for well-titled topic branches!

## Copyright

Copyright (c) 2015 Participatory Politics Foundation and DataMade. Released under the [MIT License](https://github.com/datamade/councilmatic-starter-template/blob/master/LICENSE).
