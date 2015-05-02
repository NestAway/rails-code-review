# Rails Code Review

An evolving set of guidelines & supporting reasons to consider when code reviewing Ruby on Rails apps.

<!-- MarkdownTOC depth=0 autolink=true bracket=round -->

- [If Statements](#if-statements)
  - [Avoid `== boolean`](#avoid--boolean)
- [Security](#security)
  - [Keep Rails Vulnerabilities Patched](#keep-rails-vulnerabilities-patched)
  - [Purge Exposed Secrets](#purge-exposed-secrets)
  - [Keep Gem Versions Updated](#keep-gem-versions-updated)
  - [Force SSL](#force-ssl)
- [Gems](#gems)
  - [Group Gems Appropriately in `Gemfile`](#group-gems-appropriately-in-gemfile)
  - [Remove Unused Gems from `Gemfile`](#remove-unused-gems-from-gemfile)
  - [Track `Gemfile.lock` in Version Control](#track-gemfilelock-in-version-control)
- [Controllers](#controllers)
  - [Minimize variable scope](#minimize-variable-scope)
  - [Avoid class variables](#avoid-class-variables)
  - [Use HTTP Status Code Symbols](#use-http-status-code-symbols)
  - [Appropriate HTTP Verbs & Status Codes](#appropriate-http-verbs--status-codes)
  - [Redirect to Prevent Duplicate Form Submissions](#redirect-to-prevent-duplicate-form-submissions)
- [Presentation & Accessibility](#presentation--accessibility)
  - [Accurate Page Titles](#accurate-page-titles)
- [Database](#database)
  - [Check `schema.rb` is in Good Shape](#check-schemarb-is-in-good-shape)
- [Performance](#performance)
  - [Compress Assets](#compress-assets)
  - [Serve Assets via a CDN](#serve-assets-via-a-cdn)
  - [Consider Using a CDN for Every Request](#consider-using-a-cdn-for-every-request)
  - [Keep App Awake](#keep-app-awake)
- [Version Control](#version-control)
  - [Have a Healthy Commit History](#have-a-healthy-commit-history)
- [Documentation](#documentation)
  - [Kickstart New Developers](#kickstart-new-developers)
- [Continuous Integration](#continuous-integration)
  - [Run All Tests Frequently](#run-all-tests-frequently)
  - [Monitor & Handle Failures](#monitor--handle-failures)
- [Deployment](#deployment)
  - [Simple, Repeatable Deploys](#simple-repeatable-deploys)
  - [Minimize Manual Steps](#minimize-manual-steps)
- [Contributors](#contributors)

<!-- /MarkdownTOC -->


# If Statements

## Avoid `== boolean`

`== true` and `== false` aren't needed in `if` statements:

```ruby
if valid == true # Works

if valid # Works, more conventional

...

if valid == false # Works

if !valid # Works, more conventional
```


# Security

## Keep Rails Vulnerabilities Patched

Update Rails to the latest patch version. Edit the rails version in the Gemfile then run `bundle install`. Ensure the version of Rails you're on is still actively supported and patched.

## Purge Exposed Secrets

Have any secrets **ever** been exposed? Usually this happens when `secrets.yml` has been committed to the repository.

Any secrets that have been exposed should not be used any longer. 

An exposed `secret_token` can allow an attacker to gain command line access to your server via your Rails app.

`rake secret` can help with generating new secrets. Ensure these are updated on any affected environments (production, staging, etc.).

## Keep Gem Versions Updated

Run `bundle outdated` to check for old gem versions. Decide appropriate next steps on a gem-by-gem basis. Check the CHANGELOG for each gem. Update gems that have had security flaws fixed.

## Force SSL

```ruby
# config/environments/production.rb
config.force_ssl = true
```

The web is moving towards TLS/SSL-on everywhere. Some HTML5 features are not available if your site is not served over SSL. This is one of many steps you will want to take towards protecting the good people using your app.

(If your app is hosted on an *.herokuapp.com domain, you get to use their SSL certificate for free, i.e. https://your-app.herokuapp.com just works.)


# Gems

## Group Gems Appropriately in `Gemfile`

- Testing gems go in the `test` group, unless...
- ...they have generators, then they go in the `test` and `development` groups
- Production-specific gems (sometimes pg & rails_12factor gems) go in the `production` group

## Remove Unused Gems from `Gemfile`

Remove clutter from `Gemfile` by deleting any unused gems from the file.

## Track `Gemfile.lock` in Version Control

`Gemfile.lock` specifies the versions of gems to be used with the app across all environments. To increase consistency across environments, and reduce the likelihood for being surprised by version-related issues, then `Gemfile.lock` ought to be tracked under version control.


# Controllers

## Minimize variable scope

- Favor local variables over instance variables.
- Only use instance variables if the action renders a template that needs that variable. In controllers, instance variables are a mechanism for exposing variables to the rendered template.

Minimizing variable scope tends to help Ruby minimize its memory usage. For example, the memory allocated to local variables is usually needed for a shorter time than the memory allocated to instance variables.

Another benefit of minimizing variable scope is that variables are less likely to have naming collisions leading to unexpected buggy behaviour.

## Avoid class variables

Unless you're *absolutely certain* a class variable is what you need in a controller, you probably don't want to use one. 

All requests, from all users, that a controller receives will share the exact same class variable and any value assigned to it by one request may be overwritten by another request at an undesirable, unpredictable time.

## Use HTTP Status Code Symbols

Try to avoid magic numbers. Favor symbols for specifying statuses as they describe the purpose.

```ruby
render json: new_user, status: 201 # Avoid magic numbers
render json: new_user, status: :created  # Good!
```

status code | rails symbol
------------|-------------
200 | `:ok`
201 | `:created`
422 | `:unprocessable_entity`

More symbol values available from the individual pages indexed here: http://httpstatus.es/

## Appropriate HTTP Verbs & Status Codes

Controller actions should be routed with appropriate HTTP verbs (e.g. POST for `create`) and respond with status codes that represent the result of the request (e.g. `201 Created` for a successful `create` action API response).

## Redirect to Prevent Duplicate Form Submissions

When a form submission successfully completes, the controller action should redirect to another action. By redirecting the user to another URL, they are prevented from accidentally resubmitting the form if they hit refresh. This only applies to HTML responses, it does not apply to AJAX or API responses. AJAX and API responses should respond with the most appropriate HTTP status code (see above).


# Presentation & Accessibility

## Accurate Page Titles

Are the page titles specific to the page? Page titles are useful to humans and search engine robots.

Consider using the [`content_for`](http://api.rubyonrails.org/classes/ActionView/Helpers/CaptureHelper.html#method-i-content_for) helper or the [flutie](https://github.com/thoughtbot/flutie) gem.


# Database

## Check `schema.rb` is in Good Shape

Read through `db/schema.rb`. 

- Do the tables and columns have good names?
- Is it indexed appropriately?
- Is the database normalized (i.e. has no duplicate or redundant data)?
- If the database is denormalized, is it for a good reason (e.g. performance)?


# Performance

## Compress Assets

Serve assets (e.g. JavaScript, CSS, font, image files) compressed to clients that can handle them.

## Serve Assets via a CDN

See config option `config.asset_host`.

## Consider Using a CDN for Every Request

Consider fronting the entire app with a CDN if its requirements allow.

## Keep App Awake

If you've noticed the first request to your app after you haven't visited it in a while is slow to respond, it may be due to Heroku (and similar hosts) putting apps to sleep.

To help the host reduce their costs, they may put an app is put to sleep if a request isn't made on it for a certain length of time (1 hour on Heroku at time of writing I believe).

The app is still available, it just takes a while for that first request to be responded to while the host starts up (wakes up) the app after its nap.

To keep your app awake, it just needs to receive a request regularly. For Heroku, the New Relic add-on can be setup to do this for free, and is I suspect this is the most common solution. Alternatively, use [Kaffeine](http://kaffeine.herokuapp.com/). It's quicker to setup than New Relic and is also free.


# Version Control

## Have a Healthy Commit History

- Are commits focused, small, and made with descriptive, relevant messages?
- Are bugfix and feature branches used?


# Documentation

## Kickstart New Developers

Include up-to-date instructions for how new developers can get started with setting up, running the app and working on it.

Usually these instructions are in `README.md`.

# Continuous Integration

## Run All Tests Frequently

## Monitor & Handle Failures

# Deployment

## Simple, Repeatable Deploys

## Minimize Manual Steps

---

# Contributors

- Eliot Sykes https://eliotsykes.com/
- Your name here, contributions are welcome and easy! Just fork the GitHub repo, make your changes, then submit your pull request!
