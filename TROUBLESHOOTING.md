# Troubleshooting

## CSV files location

The `path` configuration for the CSV source plugin accepts either an absolute path or relative path from the Drupal's root folder.

The examples use a relative path and it is assumed that you place this demo module in a `modules/custom` folder. Therefore, the CSV files will be located at `modules/custom/ud_staff/sources/`.

This would look similar to:

```
.
|-- autoload.php
|-- core
|-- index.php
|-- modules
|   |-- contrib
|   |   |-- address
|   |   |-- entity_reference_revisions
|   |   |-- migrate_plus
|   |   |-- migrate_source_csv
|   |   |-- migrate_tools
|   |   `-- paragraphs
|   `-- custom
|       `-- ud_staff
|           |-- composer.json
|           |-- config
|           |-- DEVELOPMENT.md
|           |-- migrations
|           |-- README.md
|           |-- sources
|           |-- TROUBLESHOOTING.md
|           |-- ud_staff.info.yml
|           `-- ud_staff_setup
|-- profiles
|-- robots.txt
|-- sites
|-- themes
|-- update.php
`-- web.config
```

Not having the source CSV files in the proper location would trigger and error similar to:

```
[error]  Migration failed with source plugin exception: File path (modules/custom/ud_staff/sources/ud_staff.csv) does not exist.
```

If you want to place the files in a different location, you have to update the `source:path` setting in the corresponding migration files.

## Disappearing paragraphs

Paragraphs migrations are affected by a particular behavior of revisioned entities. If the host entity is deleted, and the paragraphs do not have translations, the whole paragraph gets deleted. That means that deleting a node will remove the referenced paragraphs. Because of this, if the node migration is rolled back, the paragraphs will be removed without the migrate API being aware of it. In this example, if you run a `migrate:status` command after rolling back the node migration, you will see that the paragraph migration still reports that all elements have been imported. The file migration for the images will report the same, but in that case the images will remain on the system.

In any migration project, it is common that you do rollback operations to test new field mappings or fix errors. Thus, chances are very high that you will stumble upon this behavior. Thanks to Damien McKenna for helping me understand this behavior and tracking it to the rollback() method of the `EntityReferenceRevisions` destination plugin. So, what do you do to recover the deleted paragraphs? You have to rollback both migrations: node and paragraph. And then, you have to import the two again. The following snippet shows how to do it:

* Rollback both migrations:
```
./vendor/bin/drush migrate:rollback ud_staff
./vendor/bin/drush migrate:rollback mbe_book_paragraph
```

* Import both migrations again:
```
./vendor/bin/drush migrate:import mbe_book_paragraph
./vendor/bin/drush migrate:import ud_staff
```

## Drush command not found

This module works with Drupal 8 and 9. Different versions of Drush are used for these major versions of Drupal. All the examples in this demo module use Drush 10 which works both with Drupal 8 and 9. If you are using Drush 8 the command names and aliases might be different. Execute `./vendor/bin/drush list --filter=migrate` to verify the proper commands for your version of Drush.