# Integrating October CMS with Composer and Git

*Take control of the versioning of October CMS and its themes and plugins with this configuration that integrates Composer with your October CMS project.*

Since its inception in mid-2016, [Composer](https://getcomposer.org/) has become the unchallenged standard in maintaining all libraries and functions required for
your PHP projects. Its assistance with development is unparalleled - you have full control over which dependencies you use and which versions are downloaded,
ensuring that no matter which copy of the project is in use, the environment and requirements are the same.

With full support for Composer, October CMS can be configured to install all the necessary modules, plugins and themes through Composer, allowing you to keep your
Git repository lean by only storing and tracking the files specifically for your project.

This tutorial outlines one particular configuration of October CMS that achieves the following outcomes:

- The installation and updating of all October CMS core modules through Composer.
- The installation and updating of all October CMS plugins that are made available through Composer or in a public repository.
- The installation and updating of an October CMS theme kept in a separate repository.
- Tight control over the versions of all the above through Composer's `composer.json` and `composer.lock` files.

Please note that this setup is best for experienced developers with prior knowledge of Composer and Git, who may wish to share projects with multiple developers
on multiple deployment targets. For less experienced developers, it is recommended to use [October CMS' marketplace](https://octobercms.com/plugins) to control
your project requirements.

