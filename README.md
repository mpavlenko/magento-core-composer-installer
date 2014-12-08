magento-core-composer-installer
===============================
[![Build Status](https://travis-ci.org/AydinHassan/magento-core-composer-installer.svg?branch=master)](https://travis-ci.org/AydinHassan/magento-core-composer-installer)
[![Code Coverage](https://scrutinizer-ci.com/g/AydinHassan/magento-core-composer-installer/badges/coverage.png?b=master)](https://scrutinizer-ci.com/g/AydinHassan/magento-core-composer-installer/?branch=master)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/AydinHassan/magento-core-composer-installer/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/AydinHassan/magento-core-composer-installer/?branch=master)
[![Latest Stable Version](https://poser.pugx.org/aydin-hassan/magento-core-composer-installer/v/stable.svg)](https://packagist.org/packages/aydin-hassan/magento-core-composer-installer)
[![Latest Unstable Version](https://poser.pugx.org/aydin-hassan/magento-core-composer-installer/v/unstable.svg)](https://packagist.org/packages/aydin-hassan/magento-core-composer-installer)

A Composer Plugin to manage Magento Core

This tool allows you to manage Magento Core in your project using Composer. 

Advantages:
 * Keep Magento out of your project repository
 * Allows for easy upgrades to new Magento versions
 
You require this tool in your project and require a specific version of Magento. An initial install of a 
Magento package will trigger an install to your Magento root directory. A `.gitignore` file will be automatically created
with all of the files in the Magento core package (Some are grouped, more on this later).

When a package is removed, either using `composer remove magento/magento` or manually removing from your `composer.json` file
and running `composer update` all of the files present in the version of Magento you are removing will be deleted from
your `magento-root-dir` folder. If there are custom files in any of these folders, these will not be deleted.

This allows you to install and remove the Magento core in to your project without having to commit it as the `.gitignore`
is automatically updated. Further to this, in your project root you could ignore the `.gitignore` in the `magento-root-dir`
This would mean that you can install, update & remove Magento core without any un-tracked files showing in your
repository.

Now updating Magento Core is easy, simply change your require to "magento/magento": 1.10.0 or whatever the newest version is and run `composer update`!

Compatibility
-------------

This tool works with any version of PHP >= 5.3. It is automatically tested using Travis on version PHP versions 5.3, 5.4, 5.5 & HHVM. 

Installation
------------

    $ cd magento-project
    $ composer require aydin-hassan/magento-core-composer-installer

The latest stable version will be installed.

In order for the installer to actually do anything, you will also need to require a core Magento package.
This should be a Magento release with a `composer.json` file in the root. It should be defined like so:

    {
        "name": "magento/magento",
        "description": "Magento Mirror",
        "type": "magento-core"
    }

You can create your own public or private Magento repository to host the different versions.
You should tag each version as the version it is. The `type` key is important. The Magento Core Composer Installer
will only install packages which have a type of `magento-core`.

To use the Magento package you will have to add the repository to your projects `composer.json` file:

    "repositories": [
        {
            "type": "vcs",
            "url": "git@bitbucket.org:somevendor/magento.git"
        },
    ],
    
Then you can do:

    $ composer require "magento/magento:1.9.1.0"
    
    
Overall your projects `composer.json` should look something like:

    {
        "name": "somevendor/someproject",
        "description": "Magento build for ...",
        "require-dev": {
            "phpunit/phpunit": "~4.4"
        },
        "require": {
            "aydin-hassan/magento-core-composer-installer" : "~1.0",
            "magento/magento" : "1.9.1.0"
        },
        "authors": [
            {
                "name": "Aydin Hassan",
                "email": "aydin@hotmail.co.uk"
            }
        ],
        "repositories": [
            {
                "type": "vcs",
                "url": "git@bitbucket.org:somevendor/magento.git"
            },
        ],
        "extra" {
            "magento-root-dir": "htdocs"
        }
    }
    
Also note the `magento-root-dir`, this is used to specify where you want Magento to be installed to.
    
Configuration
-------------
Configuration can be provided under the `magento-core-deploy` under the `extra` key in your root `composer.json` file

Available configuration

 * `excludes` - An array of files to not copy when installing the core, useful if local overrides are necessary, or you want to track a `.htaccess`.
 * `ignore-directories` - An array of folders to group ignores together to make the core `.gitignore` smaller.
 * `git-ignore-append` - Defaults to `false`. Whether to append to the `.gitignore` in the `magento-root-dir` folder. Otherwise it will be wiped out on each deploy.
 
Example configuration:

    {
         ...
         "extra" {
             "magento-root-dir": "htdocs",
             "magento-core-deploy" : {
                "excludes": [ 
                    ".htaccess"
                ],
                "ignore-directories": [
                    "lib/Zend"
                ],
                "git-ignore-append": false
             }
         }
    }
    
The above config will, install everything from the core package, except the file `.htaccess`, it will will group all files
under `lib\Zend` in the `htdocs\.gitignore` file. And it will wipe the `htdocs\.gitignore` every time Magento is updated or removed. 
 

### Ignore Directories

This one requires a little more explanation. Generating a `.gitignore` for every file in Magento results in a file well over 
10,000 lines. On investigation this seems to slow git commands like `git status` down quite a bit. Some issues  were taking
up to 14 seconds for me. 

In order to combat this, any files which are in a default set of folders, will not be added to the `.gitinogre`, instead
only the folder will, this greatly reduces the size of the `.gitignore`. The list of folders which are ignored by default can be 
found here: 

 * app/code/core/Mage
 * app/code/core/Zend
 * app/code/core/Enterprise
 * lib/Zend
 * lib/Varien
 * lib/Magento
 * lib/PEAR
 * lib/Mage
 * lib/phpseclib
 * lib/flex
 * lib/LinLibertineFont
 * downloader
 * js/extjs
 * js/prototype
 * js/calendar
 * js/mage
 * js/varien
 * js/tiny_mce
 * lib/Apache
 * app/code/community/Phoenix/Moneybookers
 
If you need to commit files inside these directories then you can over-ride this list by setting the `ignore-directories`
key, noted above. Your list will not be merged, it will be used instead. This is in case you want to remove one of the ignore directories.

Running the Tests
-----------------

    $ git clone git@github.com:AydinHassan/magento-core-composer-installer.git
    $ cd magento-core-composer-installer
    $ composer install
    $ ./vendor/bin/phpunit
