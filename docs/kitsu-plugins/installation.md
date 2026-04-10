# Kitsu Plugin Installation

## Prerequisites

* A running Kitsu instance with Zou API
* Admin access to the server

## Install from a Folder or Archive

Install from a local plugin folder:

```bash
zou install-plugin --path ./plugins/my-plugin
```

Or from a packaged `.zip` archive:

```bash
zou install-plugin --path ./dist/my-plugin.zip
```

Or point directly to a git repository:

```bash
zou install-plugin --path https://github.com/cgwire/kitsu-tickets.git
```

## What Happens During Installation

1. `pre_install()` lifecycle hook is called (if defined by the plugin)
2. Plugin files are copied to the Zou plugins directory
3. Alembic database migrations run automatically (creating or updating tables).
   Zou provides the Alembic boilerplate (`env.py`, `alembic.ini`,
   `script.py.mako`) — the plugin only needs to ship revision files under
   `migrations/versions/`.
4. `post_install()` lifecycle hook is called (if defined)
5. Plugin routes are registered under `/api/plugins/<plugin_id>/`

## Restart Zou

After installation, restart your Zou server so that the new routes and models
are loaded:

```bash
sudo systemctl restart zou
```

## Verify Installation

List all installed plugins to confirm:

```bash
zou list-plugins
```

If the plugin has a frontend (`frontend_studio_enabled` or
`frontend_project_enabled` in its manifest), it will appear in the Kitsu
sidebar or production menu after reloading the page.

## Update a Plugin

To update an already installed plugin, run `install-plugin` again with the new
version. Zou will run any new database migrations and call the install hooks.

```bash
zou install-plugin --path ./dist/my-plugin-v2.zip
```

## Uninstall a Plugin

```bash
zou uninstall-plugin --id my_plugin
```

This calls `pre_uninstall()` and `post_uninstall()` hooks if defined.

::: warning
Database tables created by the plugin are **not** dropped automatically during
uninstall. Remove them manually if needed.
:::
