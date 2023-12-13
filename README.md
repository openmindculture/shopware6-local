# shopware6-local

Shopware 6 demo shop playground installation using the new official [nix](https://nixos.org)-based [devenv](https://developer.shopware.com/docs/guides/installation/devenv.html) flex template using [Symfony flex recipes](https://symfony.com/doc/current/quick_tour/flex_recipes.html).

- to practice for Shopware 6 developer certification
- to test and develop [IngoSDev6CertPrep](https://github.com/openmindculture/IngoSDev6CertPrep)

This setup obsoletes earlier approaches based on [shopware/development](https://github.com/shopware/development). Some content might still need an update to match the new requirements!

## Shopware 6.x Changes not covered in the Academy Tutorials

- since 6.5 there is no psh.phar anymore. You can find the replacements in the bin folder just ./bin/watch-administration.sh and so on (source: shyim in Shopware Slack)
- see my subjective selective changelog post [Shopware changes since the 6.0 dev training videos](https://dev.to/ingosteinke/shopware-changes-since-the-60-dev-training-videos-481o) for more changes since 6.0/6.1

This main directory will ignore the old Shopware repositories
- development
- shopware

New themes and plugins below `development`
should have their own repositories below `custom`,
e.g. /development/custom/plugins/IngoSFraktalistheme

## Changelog

### 2023

- remove deprectated instances
- renamed and updated existing projects to use the new flex template
- check Shopware/Symfony documentation for changes and upcoming SW 6.6 features

### prod(uction) installation online

- installed directly on the shared host
- cloned 6.3 repository (or 6.4 beta preview)
- composer install
- clicked through web installer...
- ...issue: no admin app but white screen
- CLI: bin/build.sh
- back in admin app,
- installed demo data
- and finished basic configuration
- running demoshop!

### alternative ad-hoc setup

`./psh.phar administration:watch`

should start a demo server on localhost:8080 (untested)

## development installation on localhost

```
git clone git@github.com:shopware/development.git
cd development
git clone git@github.com:shopware/platform.git
composer install
./psh.phar docker:start
```

Enter the container like

```
docker exec -it ${app_server_id} sh
```

or

```
./psh.phar docker:ssh
```
Continue inside the container

```
./psh.phar install
```

## Log in to Shopware 6 demoshop admin

http://localhost:8000/admin 

using the initial default credentials

- User: admin
- Login: shopware

## Update local installation

Merge the latest updates from Shopware development upstream into the local repository:

```
git pull
cd platform && git pull
```

Clear and rebuild the depencies.
If necessary, run (probably both in development, and development/platform ?):

```
rm composer.lock
rm -rf vendor/*
composer install
```

You should not need to rebuild and reinstall Shopware, use the update command instead.
Inside the docker container started in `development`
using `./psh.phar docker:start`

enter the container using `./psh.phar docker:ssh`

Inside the container, type

```
./psh.phar update # don't install but use update!
```

## Troubleshooting

### Installation
If the installation terminates abnormally, it is worth to retry, after clearing caches and artifacts manually:

```
rm -fr node_modules
rm var/cache
```

### Run-Time
If the setup stops to work without any apparent reason or change, with symptoms like missing assets or throwing SQL errors, first try and rebuild the container. If the errors persist, reinstall shopware like described above.

## Local plugin development

Start / clone your plugin below
`development/custom/plugins`

Following commands _inside_ the container!

### Validate and activate

```
bin/console plugin:refresh
bin/console plugin:install --activate MyPluginOrThemeName
bin/console cache:clear
```

### Build administration app

```
./psh.phar cache
./psh.phar administration:build
```

After that, we can bundle our plugin folder as a zip archive.
If your plugin is a repository of its own, you can commit, tag, and push to GitHub
where a zip archive will be created automatically. 

### Using FroshPluginUploader to validate, bundle, and upload

Replace `feature-branch by the actual name,
otherwise the zip file will be created from `main` / `master` branch.

You can install the FroshPluginUploader parallel to `custom`:

```
git clone https://github.com/FriendsOfShopware/FroshPluginUploader
cd FroshPluginUploader
composer install
bin/pluginupload plugin:zip:dir ../custom/plugins/MyPluginOrThemeName FEATURE_BRANCH
bin/pluginupload plugin:validate ./MyPluginOrThemeName-FEATURE_BRANCH.zip
```

### Test mit dem  offiziellen Testsetup

@shyim:
> For plugin testing look into https://hub.docker.com/r/shopware/testenv

```
docker run --rm -p 80:80 -e VIRTUAL_HOST=localhost shopware/testenv:6.3
```

Falls eine neuere Version bekannt und getagged ist, kann diese hier angegeben werden.
Login in den Administrationsbereich ist in diesem Fall `demo:demo`, das entpsricht dem Default-Login von Shopware 5.
Falls der Update-Assistent ein Update auf eine neuere Shopware-Version vorschlägt, sollte dies gemacht werden.

Der Testshop befindet sich auf

http://localhost/shop/public

Der Administrationsbereich des Testshops mit Login User:demo, Passwort:demo, auf

http://localhost/shop/public/admin


### Error Logs und Profiling

Error Logs in `var/log` unterhalb des Project Root, auch außerhalb des Containers sichtbar.

Zusätzlich läuft im Browser ein `Symfony Profiler`, der in der Fußzeile
zusätzliche Informationen zu Status und Requests anzeigt.


# Container stoppen und löschen

Ist man mit den Tests fertig kann man die Container mit

```
./psh.phar docker:stop
```

stoppen und danach ggf. mit

```
docker-compose down
```

löschen.
