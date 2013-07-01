# Lita

[![Build Status](https://travis-ci.org/jimmycuadra/lita.png)](https://travis-ci.org/jimmycuadra/lita)
[![Code Climate](https://codeclimate.com/github/jimmycuadra/lita.png)](https://codeclimate.com/github/jimmycuadra/lita)
[![Coverage Status](https://coveralls.io/repos/jimmycuadra/lita/badge.png)](https://coveralls.io/r/jimmycuadra/lita)

**Lita** is a chat bot written in Ruby with persistent storage provided by [Redis](http://redis.io/). It can connect to any chat service (given that there is an [adapter](#adapters) available for it) and can have new behavior added via [handlers](#handlers). The plugin system is managed with regular RubyGems and [Bundler](http://gembundler.com/).

Automate your business and have fun with your very own robot companion.

## Why?

Lita draws much inspiration from GitHub's fantastic [Hubot](http://hubot.github.com/), but has a few key differences and strengths:

* It's written in Ruby.
* It exposes the full power of Redis rather than using it to serialize JSON.
* Is easy to develop and test plugins for with the provied [RSpec](https://github.com/rspec/rspec) extras. Lita strongly encourages thorough testing of plugins.
* It uses uses the Ruby ecosystem's standard tools (RubyGems and Bundler) for plugins.
* It's thoroughly documented.

## Dependencies

* Ruby 2.0
* Redis

## Installation

First, install the gem with `gem install lita`. This gives you access to the `lita` commmand. Run `lita help` to list available tasks.

Generate a new Lita instance by running `lita new NAME`. This will create a new directory called NAME (defaults to "lita") with a Gemfile and Lita configuration file.

## Usage

To start your Lita instance, simply run `bundle exec lita`. This will load up all the plugins (adapters and handlers) declared in your Gemfile, load any configuration you've defined (more on that later) and start the bot.

## Adapters

The core Lita gem by itself doesn't do much. To make real use of it, you'll want to install an adapter gem to allow Lita to connect to the chat service of your choice. Find the gem for the service you want to use on [the list of adapters](https://github.com/jimmycuadra/lita/wiki/Adapters), then add it to your Gemfile. For example:

``` ruby
gem "lita-hipchat"
```

Adapters will likely require some configuration to be able to connect. See the documentation for the adapter for details.

Without installing an adapter, you can use the default shell adapter to chat with Lita in your terminal. Lita doesn't respond to many messages by default, however, so you'll want to add some new behavior to Lita via handlers.

## Handlers

Handlers are gems that add new behavior to Lita. They are responsible for listening for incoming messages and responding to them appropriately. Find the handler gems you want for your bot on [the list of handlers](https://github.com/jimmycuadra/lita/wiki/Handlers), then add them to your Gemfile. For example:

``` ruby
gem "lita-karma"
```

## Configuration

To configure Lita, edit the file `lita_config.rb` generated by the `lita new` command. This is just a plain Ruby file that will be evaluated when the bot is starting up. A Lita config file looks something like this:

``` ruby
Lita.configure do |config|
  config.robot.name = "Sir Bottington"
  config.robot.mention_name = "bottington"
  config.robot.adapter = :example_chat_service
  config.adapter.username = "bottington"
  config.adapter.password = "secret"
  config.redis.host = "redis.example.com"
  config.handlers.karma.cooldown = 300
  config.handlers.google_images.safe_search = false
end
```

The main config objects are:

* `robot` - General settings for Lita.
  * `name` (String) - The display name the bot will use on the chat service. Default: `"Lita"`.
  * `mention_name` (String) - The name the bot will look for in messages to determine if the message is being addressed to it. Usually this is the same as the display name, but in some cases it may not be. For example, on HipChat, display names are required to be a first and last name, such as "Lita Bot", whereas the mention system would use a name like "LitaBot". Default: `Lita.config.robot.name`.
  * `adapter` (Symbol, String) - The adapter to load. Default: `:shell`.
  * `log_level` (Symbol, String) - The severity level of log messages to output. Valid options are, in order of severity - `:debug`, `:info`, `:warn`, `:error`, and `:fatal`. For whichever level you choose, log messages of that severity and greater will be output. Default: `:info`.
  * `admins` (Array<String>) - An array of string user IDs which tell Lita which users are considered administrators. Only these users will have access to Lita's `auth` command. Default: `nil`.
* `redis` - Options for the Redis connection. See the [Redis gem](https://github.com/redis/redis-rb) documentation.
* `adapter` - Options for the chosen adapter. See the adapter's documentation.
* `handlers` - Handlers may choose to expose a config object here with their own options. See the handler's documentation.

If you want to use a config file with a different name or location, invoke `lita` with the `-c` option and provide the path to the config file.

## Authorization

Access to commands can be allowed for only certain users by means of authorization groups. Users set as admins (by adding their user IDs to the `config.robot.admins` array in Lita's configuration) have access to two commands:

```
Lita: auth add joe committers
Lita: auth remove joe committers
```

The first command adds a user whose ID or name is "joe" to the authorization group "committers." If the group doesn't yet exist, it is created. The second command removes joe from the group. Handlers can specify that a route (a method that matches an incoming message) requires that the user sending the message be in a certain authorization group. See the section on writing handlers for more details.

## Online help

Message Lita `help` for a list of commands it knows about. You can also message it `help FOO` to list only commands beginning with FOO.

## Shell adapter

Lita ships with one adapter for use directly in the shell. Simply type text at the input to send messages, and Lita will respond with any registered handlers. The shell adapter has one configuration attribute:

* `private_chat` (Boolean) - If true, all messages will be treated as though they were sent in a private chat, so they will be considered commands even when not prefixed with the bot's name. Default: `false`.

## Writing an adapter

An adapter is a packaged as a RubyGem. The adapter is a class that inherits from `Lita::Adapter`, implements a few required methods, and is registered by calling `Lita.register_adapter(:symbol_that_identifies_the_adapter, TheAdapterClass)`.

### Example

Here is a bare bones example of an adapter for the fictious chat service, FancyChat.

``` ruby
module Lita
  module Adapters
    class FancyChat < Adapter
      # Optional. Makes the bot produce an error message and quit upon start up
      # if `config.adapter.username` or `config.adapter.password` are not set.
      require_configs :username, :password

      # Connects to the chat service and dispatches incoming messages to a
      # Lita::Robot instance.
      def run
      end

      # Sends a message from the robot to a user or room on the chat service.
      def send_messages(target, strings)
      end

      # Sets the topic for a chat room.
      def set_topic(target, topic)
      end

      # Does any clean up necessary when disconnecting from the chat service.
      def shut_down
      end
    end

    Lita.register_adapter(:fancy_chat, FancyChat)
  end
end
```

It's important to note that each adapter should employ its own thread or event mechanism so that incoming messages can still be processed even while a handler is processing a previous message.

For more detailed examples, check out the built in shell adapter or [lita-hipchat](https://github.com/jimmycuadra/lita-hipchat). Also check out the API documentation.

## Writing a handler

A handler is packaged as a RubyGem. A handler is a class that inherits from `Lita::Handler` and is registered by calling `Lita.register_handler(TheHandlerClass)`. There are two components to a handler: route definitions, and the methods that implement those routes.

To define a route, use the class method `route`:

``` ruby
route /^echo\s+(.+)/, to: :echo
```

`route` takes a regular expression that will be used to determine whether or not an incoming message should trigger the route. The keyword argument `:to` is supplied the name of the method that should be called when this route is triggered. `route` takes two additional options:

* `:command` (Boolean) - If set to true, the route will only trigger when "directed" at the robot. Directed means that it's sent via a private message, or the message is prefixed with the bot's name in some form (optionally prefixed with an @, and optionally followed by a colon or comma and white space). This prefix is stripped from the message body itself, but the `command?` method available to handlers can be used if you need to determine whether or not a message was a command after it's been routed. Default: `false`.
* `:restrict_to` (Symbol, String, Array<String, Symbol>) - Authorization groups necessary to trigger the route. The user sending the message must be a member of at least one of the supplied groups. See the section on authorization for more information. Default: `nil`.
* `:help` (Hash<String>) - A map of example invocations of the route and descriptions of what they do. These values will be used to generate the listing for the built-in "help" handler. The robot's mention name will automatically be added to the front of the example if the route is a command. Default: `{}`.

Here is an example of a route declaration with all the options:

``` ruby
route /^echo\s+(.+)/, to: :echo, command: true, restrict_to: [:testers, :committers], help => {
  "echo FOO" => "Replies back with FOO."
}
```

Each method that is called by a route takes one argument, an array of matches extracted by calling `message.scan(route_pattern)`. Handler methods have several other methods available to them to assist in performing other tasks and in most cases responding to the user who sent the message:

* `reply` - Sends one or more string messages back to the source of the original message, either a private message or a chat room.
* `redis` - A `Redis::Namespace` object which provides each handler with its own isolated Redis store, suitable for many data persistence and manipulation tasks.
* `args` - The user's message as an array of strings, as it would be parsed by `Shellwords.split`. For example, if the message was "Lita: auth add joe committers", calling `args` would return `["auth", "add", "joe", "committers"]`. This is very handy for commands that take arguments in a way similar to how a UNIX shell would work.
* `user` - A `Lita::User` object for the user who sent the message.
* `command?` - A boolean indicating whether or not the current message was directed at the robot.
* `message_body` - The full body of the user's message, as a string.

To add entries to Lita's built-in `help` command for your handler's commands, add a class method called `help` that returns a hash, where the keys are the format of the command, and the values are a description of what it does.

### Example

Here is a basic handler which simply echoes back whatever the user says.

``` ruby
module Lita
  module Handlers
    class Echo < Handler
      route /^echo\s+(.+)/, to: :echo, help: { "echo FOO" => "Echoes back FOO." }

      def echo(matches)
        reply matches
      end
    end

    Lita.register_handler(Echo)
  end
end
```
For more detailed examples, check out the built in authorization and help handlers or [lita-karma](https://github.com/jimmycuadra/lita-karma). Also check out the API documentation.

## Testing

It's a core philosophy of Lita that any handlers you write for your robot should be as thoroughly tested as any other program you would write. To make this easier, Lita ships with some handy extras for [RSpec](https://github.com/rspec/rspec) that make testing a handler dead simple.

To include Lita's RSpec extras, require "lita/rspec" and add `lita: true` to any `describe` block where you want the extras:

``` ruby
require "lita/rspec"

describe Lita::Handlers::MyHandler, lita: true do
  # ...
end
```

`Lita::RSpec` makes the following changes to make testing easier:

* `Lita.handlers` will return an array with only the class you're testing (`described_class`).
* All Redis interaction will be namespaced to a test environment and automatically cleared out before each example.
* Strings sent to `Lita::Robot#send_messages` will be pushed to an array accessible as `replies` so you can make expectations about output from the robot.
* You have access to the following cached objects set with `let`: `robot`, `source`, and `user`. Note that these objects are instances of the real classes and not test doubles.

The custom helper methods are where `Lita::RSpec` really shines. You can test routes very easily using this syntax:

``` ruby
it { routes("some message").to(:some_method) }
it { routes_command("directed message").to(:some_command_method) }
it { doesnt_route("message").to(:some_command_method) }
```

* `routes` - Sets an expectation that the given string will trigger the given method when overheard by the robot.
* `routes_command` - Sets an expectation that the given string will trigger the given method when directed at the robot, either in a private message, or by prefixing a message in a chat room with the robot's mention name.
* `doesnt_route` - An expectation that is the inverse of the one set by `routes`. Also aliased to `does_not_route`.
* `doesnt_route_command` - An expectation that is the inverse of the one set by `routes_command`. Also aliased to `does_not_route_command`.

To send a message to the robot, use `send_message` and `send_command`. Then set expectations about the contents of the `replies` array.

``` ruby
it "lets everyone know when someone is happy" do
  send_message("I'm happy!")
  expect(replies.last).to eq("Hey, everyone! #{user.name} is happy! Isn't that nice?")
end

it "greets anyone that says hi to it" do
  send_command("hi")
  expect(repliest.last).to eq("Hello, #{user.name}!")
end
```

* `send_message` - Sends the given string to the robot.
* `send_command` - Sends the given string to the robot, prefixing it with the robot's mention name.

## License

[MIT](http://opensource.org/licenses/MIT)
