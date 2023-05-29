# Islandora Open Migrate

## Introduction

Demonstration module to show off Drupal Migrate and DGI's tooling.

## Requirements

This module requires the following modules/libraries:

* [Islandora](https://github.com/Islandora/islandora)
* [DGI Migrate](https://github.com/discoverygarden/dgi_migrate/)
* [FOXML](https://github.com/discoverygarden/foxml)
* [Akubra Adapter](https://github.com/discoverygarden/akubra_adapter/)
* Migrate (part of Drupal core)

## Installation

Install as usual, see
[this](https://www.drupal.org/docs/extending-drupal/installing-modules) for
further information.

## Configuration

Using the supplied sample data, extract it on the server as follows:
```bash
cd /usr
tar -xvf cookie_fedora.tar.gz
```

This will extract our pseudo FCREPO3 data in a place that a mount would live.

Next `chown` and `chmod` the files to ensure Drupal can read them only:
```bash
sudo chown -R www-data:root /usr/local/fedora
sudo chmod -R a=rX /usr/local/fedora
```
NOTE: This is purely for demonstration purposes and I would suggest not doing this directly on a mount. Consult a
qualified dev-ops resource.

Make an entry in the Drupal settings file to allow the `akubra_adapter` to read the newly extracted data:
```php
$settings['akubra_adapter_datastream_basepath'] = '/usr/local/fedora/data/datastreamStore';
$settings['akubra_adapter_object_basepath'] = '/usr/local/fedora/data/objectStore';
```

If everything has been set up correctly if you run `drush migrate-status --group=foxml_to_islandora_open` you should see
the following:
```bash
sudo -u www-data /opt/www/drupal/vendor/bin/drush ms --group=foxml_to_islandora_open
 ---------------------------------------------------- ----------------------------------- -------- ------- ---------- ------------- ---------------
  Group                                                Migration ID                        Status   Total   Imported   Unprocessed   Last Imported
 ---------------------------------------------------- ----------------------------------- -------- ------- ---------- ------------- ---------------
  FOXML with MODS to Nodes (foxml_to_islandora_open)   islandora_open_foxml_files          Idle     40      0          40
  FOXML with MODS to Nodes (foxml_to_islandora_open)   islandora_open_stub_nodes           Idle     0       0          0
  FOXML with MODS to Nodes (foxml_to_islandora_open)   islandora_open_nodes                Idle     0       0          0
  FOXML with MODS to Nodes (foxml_to_islandora_open)   islandora_open_orig_file            Idle     0       0          0
  FOXML with MODS to Nodes (foxml_to_islandora_open)   islandora_open_store_source_foxml   Idle     0       0          0
  FOXML with MODS to Nodes (foxml_to_islandora_open)   islandora_open_orig_media           Idle     0       0          0
 ---------------------------------------------------- ----------------------------------- -------- ------- ---------- -----
```

For ease of running use the included helper scripts from within `dgi_migrate`. Create a folder to house log outputs as
well as the environment file.

```bash
cd /opt/www/drupal
mkdir islandora_open_migrations
cd islandora_open_migrations
cp /opt/www/drupal/modules/contrib/dgi_migrate/scripts/env.sample .env
```

Open the .env in your favorite text editor and enter values for both the `MIGRATION_GROUP` and `URI`:
```bash
# ===
# MIGRATION_GROUP: The migration group on which to operate.
#
# No default, as there is nothing sane to provide as a default.
# ---
MIGRATION_GROUP=foxml_to_islandora_open

# ===
# WEB_USER: The user as whom to invoke drush; must have access to JWT keys and
# the like.
# ---
#WEB_USER=www-data

# ===
# URI: The URI of the target site. Must be externally dereferenceable, as this
# is likely used to generate externally links, and derivatives and the like.
# ---
# NOTE: URI has no default, and _must_ be provided by all sites.
URI=http://tophat-i9manage-dev.dev.dgi
```

The migration is now ready to be run by executing the following:
```bash
bash /opt/www/drupal/modules/contrib/dgi_migrate/scripts/migration.sh /opt/www/islandora_open_migrations
```

The output of the migration can be viewed in the logs that are generated in the directly made and also includes any
messages from within the migration's tables in JSON format.

```bash
ubuntu@tophat-i9manage-dev:/opt/www/drupal/islandora_open_migrations/01-messages$ ls -lah
total 16K
drwxrwsr-x 2 www-data www-data 4.0K May 29 14:05 .
drwxrwsr-x 3 ubuntu   www-data 4.0K May 29 14:05 ..
-rw-rw-r-- 1 www-data www-data    0 May 29 14:05 islandora_open_foxml_files.json
-rw-rw-r-- 1 www-data www-data 7.2K May 29 14:05 islandora_open_nodes.json
-rw-rw-r-- 1 www-data www-data    0 May 29 14:05 islandora_open_orig_file.json
-rw-rw-r-- 1 www-data www-data    0 May 29 14:05 islandora_open_orig_media.json
-rw-rw-r-- 1 www-data www-data    0 May 29 14:05 islandora_open_store_source_foxml.json
-rw-rw-r-- 1 www-data www-data    0 May 29 14:05 islandora_open_stub_nodes.json
```

Lastly migrations can be rolled back by doing the following:
```bash
bash /opt/www/drupal/modules/contrib/dgi_migrate/scripts/rollback.sh /opt/www/islandora_open_migrations
```

## License

[GPLv3](http://www.gnu.org/licenses/gpl-3.0.txt)

