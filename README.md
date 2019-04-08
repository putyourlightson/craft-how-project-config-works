# How Project Config Works

> *Compiled by Ben Croker of [PutYourLightsOn](https://putyourlightson.com/).*

As of Craft 3.1, *Project Config* is a thing. Here is a run down of how it works.

## Overview

- Project config is *always on*. 
- Project config is responsible for the site schema (see below).
- Project config is stored in the *config* column of the *Info* table in the database as a serialized array.
- Project config is backed up automatically and stored in `.yaml` files in `storage/config-backups`.
- Project config data should only be editable by admins.

## The Project Config File

- Enabling `useProjectConfigFile` tells Craft to store project config in a `config/project.yaml` file and use that version as the single source of truth.
- Craft monitors the `project.yaml` file for changes and if it detects any then syncs them to project config in the database.
- As a best practice, site schema changes should only be made in the environment in which the `project.yaml` file is version controlled and deployed from.
- The value stored in the *config* column of the *Info* table is still what is loaded on each regular request.

## Caveats

- If Craft detects that `project.yaml` has changed but that the versions of Craft and plugins in the file are incompatible with what’s actually installed then access to the control panel will be denied. Running `composer install` should resolve any discrepancies.
- Use environment variables to store sensitive information in settings so as to prevent them being stored in plaintext in project config.

## Plugins

- Plugins that manipulate the site schema should do so using service APIs or the *ProjectConfig* service instead of directly in the database.
- Plugins that store references to site schema components in project config should store the UIDs (universally unique identifiers) instead of the IDs of those components.

## Plugin Migrations

- Plugin migrations that update the site schema (including their own settings) should do so using the *Plugins* service or their own service instead of directly in the database.

- All plugins are updated before applying all the other `project.yaml` changes, so if you update project config file from a plugin migration, the end result is that Craft thinks that the `project.yaml` file is synced already, so the other changes never get applied.

- A schema version check should to be made against the `project.yaml` file, not the database, because you're really trying to prevent potentially breaking `project.yaml file`. 

  ```
  if (version_compare($schemaVersion, '<NewSchemaVersion>', '<')) {
  	// Make the config changes here...
  }
  ```

- By setting the `muteEvents` flag on the `ProjectConfig` service to `true`, you can prevent changes from `project.yaml` to triggering events. This is useful if you want to modify something in plugin settings, for example. The best way to do it is to set the flag to `true`, make your changes to *both* project config and then database and then (FOR THE LOVE OF GOD) set the flag back to `false`. The reason behind this is if you have a `project.yaml` file with incoming changes in the same git-pull or deploy where a new version of a plugin is coming in that needs to modify its settings, for example.

## Site Schema

- Asset volumes and image transforms
- Category groups 
- Email settings
- Fields and field groups
- Global set settings
- Matrix block types
- Plugin settings
- Routes defined in *Settings → Routes*
- Sections and entry types
- Sites and site groups
- System settings
- Tag groups
- User groups and settings
