# Updating Kitsu

This guide explains how to upgrade a self-hosted Kitsu instance to the latest
release. It covers both the Zou API and the Kitsu web app.

## Updating Zou

### Update package

First, you have to upgrade the zou package:

```bash
sudo /opt/zou/zouenv/bin/python -m pip install --upgrade zou
```

### Update database schema

Then, you need to upgrade the database schema:

```bash
DB_PASSWORD=mysecretpassword /opt/zou/zouenv/bin/zou upgrade-db
```

### Restart the Zou service

Finally, restart the Zou service:

```bash
sudo systemctl restart zou zou-events
```

That's it! Your Zou instance is now up to date.

*NB: Make it sure by getting the API version number from `https://myzoudomain.com/api`.*

## Updating Kitsu

To update Kitsu, update the files:

```bash
sudo rm -rf /opt/kitsu/dist
sudo mkdir /opt/kitsu/dist
curl -L -o /tmp/kitsu.tgz $(curl -v https://api.github.com/repos/cgwire/kitsu/releases/latest | grep 'browser_download_url.*kitsu-.*.tgz' | cut -d : -f 2,3 | tr -d \")
sudo tar xvzf /tmp/kitsu.tgz -C /opt/kitsu/dist/
rm /tmp/kitsu.tgz
```
