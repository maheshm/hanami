# Lotus

A complete web framework for Ruby

## Frameworks

Lotus combines together small but yet powerful frameworks:

* [**Lotus::Utils**](https://github.com/lotus/utils) - Ruby core extentions and class utilities
* [**Lotus::Router**](https://github.com/lotus/router) - Rack compatible HTTP router for Ruby
* [**Lotus::Validations**](https://github.com/lotus/validations) - Validation mixin for Ruby objects
* [**Lotus::Model**](https://github.com/lotus/model) - Persistence with entities, repositories and data mapper
* [**Lotus::View**](https://github.com/lotus/view) - Presentation with a separation between views and templates
* [**Lotus::Controller**](https://github.com/lotus/controller) - Full featured, fast and testable actions for Rack

All those components are designed to be used independently from each other. If your aren't familiar with them, please take time to go through their READMEs.

## Status

[![Gem Version](https://badge.fury.io/rb/lotusrb.png)](http://badge.fury.io/rb/lotusrb)
[![Build Status](https://secure.travis-ci.org/lotus/lotus.png?branch=master)](http://travis-ci.org/lotus/lotus?branch=master)
[![Coverage](https://coveralls.io/repos/lotus/lotus/badge.png?branch=master)](https://coveralls.io/r/lotus/lotus)
[![Code Climate](https://codeclimate.com/github/lotus/lotus.png)](https://codeclimate.com/github/lotus/lotus)
[![Dependencies](https://gemnasium.com/lotus/lotus.png)](https://gemnasium.com/lotus/lotus)
[![Inline docs](http://inch-ci.org/github/lotus/lotus.png)](http://inch-ci.org/github/lotus/lotus)

## Contact

* Home page: http://lotusrb.org
* Mailing List: http://lotusrb.org/mailing-list
* API Doc: http://rdoc.info/gems/lotusrb
* Bugs/Issues: https://github.com/lotus/lotus/issues
* Support: http://stackoverflow.com/questions/tagged/lotus-ruby
* Chat: https://gitter.im/lotus/chat

## Rubies

__Lotus__ supports Ruby (MRI) 2+

## Installation

```shell
% gem install lotusrb
```

## Usage

```shell
% lotus new bookshelf
% cd bookshelf && bundle
% bundle exec lotus server # visit http://localhost:2300
```

## Architectures

Lotus is a modular web framework.
It scales from **single file HTTP endpoints** to **multiple applications in the same Ruby process**.

Unlike other Ruby web frameworks, Lotus has **flexible conventions for code structure**.
Developers can arrange the layout of their projects as they prefer.
There is a suggested architecture that can be easily changed with a few settings.

Lotus encourages the use of Ruby namespaces. This is based on the experience of working on dozens of projects.
By using Ruby namespaces, as your code grows it can be split with less effort. In other words, Lotus is providing gentle guidance for **avoid monolithic applications**.

Lotus has a smart **mechanism of duplication of its frameworks**.
It allows multiple copies of the framework and multiple applications to run in the **same Ruby process**.
In other words, Lotus applications are ready to be split into smaller parts but these parts can coexist in the same heap space.

All this adaptability can be helpful to bend the framework for your advanced needs, but we recognize the need of a guidance in standard architectures.
For this reason Lotus is shipped with code generators.


### _Container_ architecture

**TL;DR: Develop your application like a gem. Implement use cases in `lib/`. Use one or more Lotus applications in `apps/`.**

This is the default architecture.
When your are about to start a new project use it.

The core of this architecture lives in `lib/`, where developers should build features **independently from the delivery mechanism**.

Imagine you are building a personal finance application, and you have a feature called _"register expense"_. This functionality involves `Money` and `Expense` Ruby objects and the need of persisting data into a database. You can have those classes living in `lib/pocket/money.rb` and `lib/pocket/expense.rb` and use [Lotus::Model](https://github.com/lotus/model) to persist them.

It has few but important advantages:

* **Code reusability.** You can register an expense from the Web UI or from a JSON API, which can be different Lotus _slices_ or simple Rack based apps.
* **Decoupled components.** The core of your application depends only on a few gems and it doesn't need to worry about the Web/HTTP/Console/Background jobs.
* **Applications are built like a gem**, this ease the process of package them and share between projects, without the need of carry on a lot of dependencies.

Multiple Lotus applications are located under the `apps/` directory.


When a project is new, to move fast, a team wants to ship a single deliverable in production.
Once the code grows, they may want to decide to extract microservices. However, this isn't always possible or easy, because of the high coupling between components.

Lotus comes to the rescue, by helping teams to build products with _slices_.
A _slice_ is a web application that serves for a single purpose: user facing application, administration backend, JSON API, dashboard for profiling etc. To make a Rails analogy, it's kinda equivalent to an _engine_. The only difference is that a _slice_ is a self contained Rack application that can be easily extracted.

Here's the name _**container**_: a Lotus _"shell"_ that can run multiple micro applications in the same process.

```shell
% lotus new pocket --arch=container
% lotus new pocket # --arch=container is the default
```

Read more about this [architecture](https://github.com/lotus/lotus/wiki/Container-architecture).

### _Application_ architecture

_upcoming_

### _Micro_ architecture

_upcoming_

## Conventions

* Lotus expects controllers, actions and views to have a specific pattern (see [Configuration](#configuration) for customizations)
* All the commands must be run from the root of the project. If this requirement cannot be satisfied, please hardcode the path with `Configuration#root`.
* The template name must reflect the name of the corresponding view: `Bookshelf::Views::Dashboard::Index` for `dashboard/index.html.erb`.
* All the static files are served by the internal Rack middleware stack.
* The application expects to find static files under `public/` (see `Configuration#assets`)
* If the public folder doesn't exist, it doesn't serve static files.

## Non-Conventions

* The application structure can be organized according to developer needs.
* No file-to-name convention: modules and classes can live in one or multiple files.
* No autoloading paths. They must be explicitly configured.

## Configuration

<a name="configuration"></a>

A Lotus application can be configured with a DSL that determines its behavior.

```ruby
require 'lotus'

module Bookshelf
  class Application < Lotus::Application
    configure do
      ########################
      # BASIC CONFIGURATIONS #
      ########################

      # Determines the root of the application (optional)
      # Argument: String, Pathname, defaults to Dir.pwd
      #
      root 'path/to/root' # or __root__

      # The relative load paths where the application will recursively load the code (mandatory)
      # Argument: String, Array<String>, defaults to empty set
      #
      load_paths << [
        'controllers',
        'views'
      ]

      # Handle exceptions with HTTP statuses (true) or don't catch them (false)
      # Argument: boolean, defaults to true
      #
      handle_exceptions true

      #######################
      # HTTP CONFIGURATIONS #
      #######################

      # The route set (mandatory)
      # Argument: Proc with the routes definition
      #
      routes do
        get '/', to: 'home#index'
      end

      # The route set (mandatory) (alternative usage)
      # Argument: A relative path where to find the routes definition
      #
      routes 'config/routes'

      # URI scheme used by the routing system to generate absolute URLs (optional)
      # Argument: A string, default to "http"
      #
      scheme 'https'

      # URI host used by the routing system to generate absolute URLs (optional)
      # Argument: A string, default to "localhost"
      #
      host 'bookshelf.org'

      # URI port used by the routing system to generate absolute URLs (optional)
      # Argument: An object coercible to integer, default to 80 if the scheme is http and 443 if it's https
      # This SHOULD be configured only in case the application listens to that non standard ports
      #
      port 2323

      # Toggle cookies (optional)
      # Argument: A [`TrueClass`, `FalseClass`], default to `FalseClass`.
      #
      cookies true

      # Toggle sessions (optional)
      # Argument: Symbol the Rack session adapter
      #           A Hash with options
      #
      sessions :cookie, secret: ENV['SESSIONS_SECRET']

      # Default format for the requests that don't specify an HTTP_ACCEPT header (optional)
      # Argument: A symbol representation of a mime type, default to :html
      #
      default_format :json

      # Rack middleware configuration (optional)
      #
      middleware.use Rack::Protection

      # HTTP Body parsers (optional)
      # Parse non GET responses body for a specific mime type
      # Argument: Symbol, which represent the format of the mime type (only `:json` is supported)
      #           Object, the parser
      #
      body_parsers :json, MyXMLParser.new

      ###########################
      # DATABASE CONFIGURATIONS #
      ###########################

      # Configure a database adapter (optional)
      # Argument: A Hash with the settings
      #           type: Symbol, :file_system, :memory and :sql
      #           uri:  String, 'file:///db/bookshelf'
      #                         'memory://localhost/bookshelf'
      #                         'sqlite:memory:'
      #                         'sqlite://db/bookshelf.db'
      #                         'postgres://localhost/bookshelf'
      #                         'mysql://localhost/bookshelf'
      #
      adapter type: :file_system, uri: ENV['DATABASE_URL']

      # Configure a database mapping (optional)
      # Argument: Proc
      #
      mapping do
        collection :users do
          entity     User
          repository UserRepository

          attribute :id,   Integer
          attribute :name, String
        end
      end

      # Configure a database mapping (optional, alternative usage)
      # Argument: A relative path where to find the mapping definitions
      #
      mapping 'config/mapping'

      ############################
      # TEMPLATES CONFIGURATIONS #
      ############################

      # The layout to be used by all the views (optional)
      # Argument: A Symbol that indicates the name, default to nil
      #
      layout :application # Will look for Bookshelf::Views::ApplicationLayout

      # The relative path where to find the templates (optional)
      # Argument: A string with the relative path, default to the root of the app
      #
      templates 'templates'

      #########################
      # ASSETS CONFIGURATIONS #
      #########################

      # Specify sources for assets (optional)
      # Argument: String, Array<String>, defaults to 'public'
      #
      assets << [
        'public',
        'vendor/assets'
      ]

      # Enabling serving assets (optional)
      # Argument: boolean, defaults to false
      #
      serve_assets true

      #############################
      # FRAMEWORKS CONFIGURATIONS #
      #############################

      # Low level configuration for Lotus::Model (optional)
      # The given block will be yielded by `Lotus::Model::Configuration`.
      # See the related documentation
      # Argument: Proc
      #
      model.prepare do
        # ...
      end

      # Low level configuration for Lotus::View (optional)
      # The given block will be yielded every time `Lotus::View` is included.
      # This is helpful to share logic between views
      # See the related documentation
      # Argument: Proc
      #
      view.prepare do
        include MyCustomRoutingHelpers # included by all the views
      end

      # Low level configuration for Lotus::Controller (optional)
      # Argument: Proc
      controller.prepare do
        include Authentication # included by all the actions
        before :authenticate!   # run auth logic before each action
      end
    end

    ###############################
    # ENVIRONMENTS CONFIGURATIONS #
    ###############################

    configure :development do
      # override the general configuration only for the development environment
      handle_exceptions false
      serve_assets      true
    end

    configure :test do
      # override the general configuration only for the test environment
      host 'test.host'
    end
  end
end
```

## Command line

Lotus provides a few command line utilities:

### Server

It looks at the `config.ru` file in the root of the application, and starts the Rack server defined in your `Gemfile` (eg. puma, thin, unicorn). It defaults to WEBRick.

It supports **code reloading** feature by default, useful for development purposes.

```shell
% bundle exec lotus server
```

### Console

It starts a REPL, by using the engine defined in your `Gemfile`. It defaults to IRb. **Run it from the root of the application**.

```shell
% bundle exec lotus console
```

It supports **code reloading** via the `reload!` command.

### Routes

It prints the routes defined by the Lotus application(s).

```shell
% bundle exec lotus routes
```

### Version

It prints the current Lotus version.

```shell
% bundle exec lotus version
```

## Contributing

1. Fork it ( https://github.com/lotus/lotus/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request

## Versioning

__Lotus__ uses [Semantic Versioning 2.0.0](http://semver.org)

## Copyright

Copyright 2014 Luca Guidi – Released under MIT License
