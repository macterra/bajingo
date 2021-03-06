[![NPM](https://nodei.co/npm/jingo.png?compact=true)](https://npmjs.org/package/jingo)

**Warning: the current version of Jingo (1.x.x) is relatively new and it has gone through some major rewrites. If you find yourself having problems with this version, you can still use the previous stable [version 0.6.1](https://github.com/claudioc/jingo/releases/tag/v0.6.1). If this is the case, please take a minute and fill out one [issue](https://github.com/claudioc/jingo/issues). And don't forget to take a look at the [list of changes](https://github.com/claudioc/jingo/blob/master/ChangeLog.md)!**

JINGO
=====

A **git based** _wiki engine_ written for **node.js**, with a decent design, a search capability and a good typography.

![Screenshot](https://dl.dropboxusercontent.com/u/152161/jingo/ss1.png)

<!-- toc -->

Table of contents
-----------------

  * [Introduction](#introduction)
  * [Features](#features)
  * [Installation](#installation)
  * [Authentication and Authorization](#authentication-and-authorization)
  * [Common problems](#common-problems)
  * [Known limitations](#known-limitations)
  * [Customization](#customization)
  * [Editing](#editing)
  * [Configuration options reference](#configuration-options-reference)

<!-- toc stop -->

Introduction
-------------

The aim of this wiki engine is to provide an easy way to create a centralized documentation area for people used to work with **git** and **markdown**. It should fit well into a development team without the burden to have to learn a complex and usually overkill application.

Jingo is very much inspired by (and compatible with) the github own wiki system [Gollum](https://github.com/gollum/gollum), but it tries to be more a stand-alone and complete system than gollum is.

Think of jingo as "the github wiki, without github but with more features". "Jingo" means "Jingo is not Gollum" for more than one reason.

There is a demo server running at http://jingo.cica.li:6067/wiki/home

![Screenshot](https://dl.dropboxusercontent.com/u/152161/jingo/ss2.png)

Features
--------

- No database: uses a git repository as the document archive
- No user management: authentication is provided via with Google logins or simple, one-user login
- Markdown for everything, [github flavored](http://github.github.com/github-flavored-markdown/)
- Uses [Codemirror](http://codemirror.net/) or [Markitup](http://markitup.jaysalvat.com/home/) as the markup editor, with a nice (ajax) preview (see the `features` key in the config file)
- Provides a "distraction free", almost full screen editing mode
- Compatible with a wiki created with the [Gollum](https://github.com/github/gollum) wiki
- Revision history for all the pages
- Show differences between document revisions
- Paginated list of all the pages, with a quick way to find changes between revisions
- Search through the content _and_ the page names
- Layout accepts custom sidebar and footer
- Gravatar support
- Can include IFRAMEs in the document (es: embed a Google Drive document)
- Can use custom CSS and JavaScript scripts
- White list for authorization on page reading and writing
- Detects unwritten pages (will appear in red)
- Automatically push to a remote
- Mobile friendly (based on Bootstrap 3.x)
- Quite configurable, but also works out of the box

For code syntax highlighting, Jingo uses the `node-syntaxhighlighter` module. For the list of supported languages, please refer to [this page](https://github.com/thlorenz/node-syntaxhighlighter/tree/master/lib/scripts).

![Screenshot](https://dl.dropboxusercontent.com/u/152161/jingo/ss3.png)

Installation
------------

`npm install jingo` or download/clone the whole thing and run "npm install".

Jingo needs a config file and to create a sample config file, just run `jingo -s`, redirect the output on a file and then edit it (`jingo -s > config.yaml`). The config file contains all the available configuration keys. Be sure to provide a valid server hostname (like wiki.mycompany.com) for Google Auth to be able to get back to you.

This document contains also [the reference](#configuration-options-reference) for all the possible options.

If you define a `remote` to push to, then jingo will automatically issue a push to that remote every `pushInterval` seconds. You can also specify a branch using the syntax "remotename branchname". If you don't specify a branch, Jingo will use `master`. Please note that before the `push`, a `pull` will also be issued (at the moment Jingo will not try to resolve conflicts, though).

The basic command to run the wiki will then be

`jingo -c /path/to/config.yaml`

Before running jingo you need to initialise its git repository somewhere (`git init` is enough).

If you define a remote to push to, be sure that the user who'll push has the right to do so.

If your documents reside in subdirectory of your repository, you need to specify its name using the `docSubdir` configuration option. The `repository` path _must_ be an absolute path pointing to the root of the repository.

If you want your wiki server to only listen to your `localhost`, set the configuration key `localOnly` to true.

![Screenshot](https://dl.dropboxusercontent.com/u/152161/jingo/ss4.png)

Authentication and Authorization
--------------------------------

You can enable two authentication strategies: _Google logins (OAuth2)_ or a simple, locally verified username/password credentials match (called "alone"). If you use the _alone_ method, you can have _only one user_ accessing the wiki (thus the name).

The _Google Login_ uses OAuth 2 and that means that on a fresh installation you need to get a `client id` and a `client secret` from Google and put those informations in the configuration file.

Follow these instructions:

* Open the [Google developer console](https://code.google.com/apis/console/)
* Create a new project (you can leave the _Project id_ as it is). This will take a little while
* Open the _Consent screen_ page and fill in the details (particularly, the _product name_)
* Now open _APIs & auth_ => _Credentials_ and click on _Create new client id_
* Here you need to specify the base URL of your jingo installation. Google will fill in automatically the other field
  with a `/oauth2callback` URL, which is fine
* Now you need to copy the `Client ID` and `Client secret` in your jingo config file in the proper places

The _alone_ method uses a `username`, a `passwordHash` and optionally an `email`. The password is hashed using a _non salted_ SHA-1 algorithm, which makes this method not the safest in the world but at least you don't have a clear text password in the config file. To generate the hash, use the `--hash-string` program option: once you get the hash, copy it in the config file.

You can enable both authentication options at the same time. The `alone` is disabled by default.

The _authorization_ section of the config file has two keys: `anonRead` and `validMatches`. If the `anonRead` is true, then anyone who can access the wiki can read anything.

If anonRead is false you need to authenticate also for reading and then the email of the user _must_ match at least one of the regular expressions provided via validMatches, which is a comma separated list. There is no "anonWrite", though. To edit a page the user must be authenticated.

The authentication is mandatory to edit pages from the web interface, but jingo works on a git repository; that means that you could skip the authentication altogether and edit pages with your editor and push to the remote that jingo is serving.

Common problems
---------------

Sometimes upgrading your version of node.js could break the `iconv` module. Try updating it with `npm install iconv`.

Known limitations
-----------------

- The authentication is mandatory (no anonymous writing allowed). See also issue #4
- The repository is "flat" (no directories or namespaces)
- Authorization is only based on a regexp'ed white list with matches on the user email address
- There is one authorization level only (no "administrators" and "editors")
- At the moment there is no "restore previous revision", just a revision browser
- No scheduled pull or fetch from the remote is provided (because handling conflicts would be a bit too... _interesting_)

Please note that at the moment it is quite "risky" to have someone else, other than jingo itself, have write access to the remote / branch jingo is pushing to. The push operation is supposed to always be successfull and there is no pull or fetch. You can of course manage to handle pull requests yourself.

Customization
-------------

You can customize jingo in four different ways:

- add a left sidebar to every page: just add a file named `_sidebar.md` containing the markdown you want to display to the repository. You can edit or create the sidebar from jingo itself, visiting `/wiki/_sidebar` (note that the title of the page in this case is useless)
- add a footer to every page: the page you need to create is "_footer.md" and the same rules for the sidebar apply
- add a custom CSS file, included in every page as the last file. The name of the file must be `_style.css` and must reside in the repository. It is not possible to edit the file from jingo itself
- add a custom JavaScript file, included in every page as the last JavaScript file. The name of the file must be `_script.js` and must reside in the repository. It is not possible to edit the file from jingo itself

All those files are cached (thus, not re-read for every page load, but kept in memory). This means that for every modification in _style.css and _script.js you need to restart the server (sorry, working on that).

This is not true for the footer and the sidebar but ONLY IF you edit those pages from jingo (which in that case will clear the cache by itself).

Editing
-------

To link to another Jingo wiki page, use the Jingo Page Link Tag.

    [[Jingo Works]]

The above tag will create a link to the corresponding page file named `jingo-works.md`. The conversion is as follows:

  1. Replace any spaces (U+0020) with dashes (U+002D)
  2. Replace any slashes (U+002F) with dashes (U+002D)

If you'd like the link text to be something that doesn't map directly to the page name, you can specify the actual page name after a pipe:

    [[How Jingo works|Jingo Works]]

The above tag will link to `Jingo-Works.md` using "How Jingo Works" as the link text.

Configuration options reference
-------------------------------

####application.title

  This will be showed on the upper left corner of all the pages, in the main toolbar

####application.repository

  Absolute path for your documents repository (mandatory).

####application.docSubdir

  If your documents reside inside a directory of the repository, specify its name here.

####application.remote

  This is the name of the remote you want to push/pull to/from (optional). You can also specify a specific branch using the syntax “remotename branchname”. If you don’t specify a branch, Jingo will use master.

####application.pushInterval

  Jingo will try to push to the remote (if present) every XX seconds (defaults to 30)

####application.secret

  Just provide a string to be used to crypt the session cookie

####application.git

  You can specify a different git binary, if you use more than one in your system

####application.skipGitCheck

  Jingo will refuse to start if it founds a version of git which is known to be problematic. You can still force it to start anyway, providing `true` as the value for this option

####application.loggingMode

  Specifies how verbose the http logging should be. Accepts numeric values: `0` for no logging at all, `1` for the a combined log and `2` for a coincise, coloured log (good for development). Default is `1`.

####application.pedanticMarkdown

  Boolean, defaults to `true` (was `false` in jingo < 1.1.0)

  The markdown module we use (Marked) tries to overcome some "obscure" problems with the original Perl markdown parser by default. These produces some problems when rendering HTML embedded in a markdown document (see also issue https://github.com/claudioc/jingo/issues/48. By default we now want to use the original parser and not the modified one (pedantic: true).

  With this option you can revert this decision if for some reason your documents are not rendered how you like.

####authentication.google.enabled

  Boolean, defaults to `true`

####authentication.google.clientId
####authentication.google.clientSecret

  Values required for Google OAuth2 authentication. Refer to a previous section of this document on how to set them up.

####authentication.alone.enabled

  Boolean, defaults to `false`

####authentication.alone.username

  Provide any username you like, as a string

####authentication.alone.passwordHash

  Use an hash of your password. Create the hash with `jingo -# yourpassword`

####authentication.alone.email

  If you want to use Gravatar, provide your gravatar email here.

####features.markitup

  Boolean, whether to enable Markitup or not (default false)

####features.codemirror

  Boolean, whether to enable Codemirror or not (default true)

  Please note that you cannot enable both editors at the same time.

####server.hostname

  This is the hostname used to build the URL for your wiki pages. The reason for these options to exist is due to the need for the OAuth2 authentication to work (it needs an endpoint to get back to)

####server.port

  Jingo will listen on this port

####server.localOnly

  Set this to `true` if you want to accept connection only _from_ localhost (default false)

####server.baseUrl

  Not used anymore (built with "//" + hostname + ":" + port)

####authorization.anonRead

  Boolean to enable/disable the anonymous access to the wiki content (default true)

####authorization.validMatches

  This is a regular expression which will be used against the user email account to be able to access the wiki. By default all google powered emails are OK, but you can for example set a filter so that only the hostname from your company will be allowed access.

####pages.index

  Defines the page name for the index of the wiki (default is "Home")

####pages.title.fromFilename

  If this is true, the title of each page will be derived from the document's filename. This is how Gollum works and from Jingo 1.0 this is now the default (default true). An important consequence of this behavior is that now Jingo is able _to rename_ documents (according to the new name it will be eventually given to), while previously it was impossible.

####pages.title.fromContent

  If this is true, the title of the document will be part of the document itself (the very first line). This is the default behavior of Jingo < 1.0 and the default is now false. If you have an old installation of Jingo, please set this value to true and `fromFilename` to false.

####pages.title.asciiOnly

  If this is set to true, Jingo will convert any non-Ascii character present in the title of the document to an ASCII equivalent (using iconv), when creating the filename of the document. Default was true for Jingo < 1.0 while for Jingo >= 1.0 the default is false

####pages.title.lowercase

  If this is set to true, Jingo will lowercase any character of the title when creating the filename. Default was true for Jingo < 1.0 while for Jingo >= 1.0 the default is false

####pages.title.itemsPerPage

  This defines how many page item to show in the "list all page" page. Keep this value as low as possible (default to 10) for performance reasons.

####customizations.sidebar

  Defines the name for the _sidebar_ component. Defaults to '_sidebar'. Please note that if you need to use a wiki coming from Github, this name should be set to '_Sidebar'

####customizations.footer

  Defines the name for the _footer_ component. Defaults to '_footer'. Please note that if you need to use a wiki coming from Github, this name should be set to '_Footer'

####customizations.style

  Defines the name for the customized _style_ CSS component. Defaults to '_style'.

####customizations.script

  Defines the name for the customized _script_ JavaScript component. Defaults to '_script'.
