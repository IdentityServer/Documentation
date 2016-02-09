Documentation for IdentityServer3
============================================

site: https://identityserver.github.io/Documentation/

## How to Build
This assumes you already have Ruby and [Bundler](http://bundler.io/) installed.

* Navigate to the project's source code
* Run `bundle install` to install the required modules
* Modify the _config.yml file to change the URL to `url: http://localhost:4000`
 * This will allow the relative links to work as expected.
 * **NOTE:** Do not commit this change.
* Run `budle exec jekyll serve` to serve the web site.