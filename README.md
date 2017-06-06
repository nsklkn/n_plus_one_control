# N + 1 Control

RSpec and Minitest matchers to prevent N+1 queries problem.

### Why yet another gem to assert DB queries?

Unlike other libraries (such as [db-query-matchers](https://github.com/brigade/db-query-matchers), [rspec-sqlimit](https://github.com/nepalez/rspec-sqlimit), etc), with `n_plus_one_control` you don't have to specify exact expectations to control your code behaviour (e.g. `expect { subject }.to query(2).times`).

Such expectations are rather hard to maintain, 'cause there is a big chance of adding more queries, not related to the system under test.

NPlusOneControl works differently. It evaluates the code under consideration several times with different scale factors to make sure that the number of DB queries behaves as expected (i.e. O(1) instead of O(N)).

### Why not just use [`bullet`](https://github.com/flyerhzm/bullet)?

Of course, it's possible to use Bullet in tests (see more [here](https://evilmartians.com/chronicles/fighting-the-hydra-of-n-plus-one-queries)), but it's not a _silver bullet_: there can be both false positives and true negatives.

This gem was born after I've found myself not able to verify with a test yet another N+1 problem.

## Installation

Add this line to your application's Gemfile:

```ruby
group :test do
  gem 'n_plus_one_control'
end
```

And then execute:

    $ bundle

## Usage

### RSpec

Example:

```ruby
# Wrap example into a context with :n_plus_one tag
context "N+1", :n_plus_one do
  # Define `populate` callbacks which is responsible for data
  # generation (and whatever else).
  #
  # It accepts one argument – the scale factor (read below)
  populate { |n| create_list(:post, n) }

  specify do
    expect { get :index }.to perform_constant_number_of_queries
  end
end
```

**NOTE:** do not use memoized values within the expectation block!

```ruby
# BAD – won't work!

subject { get :index }

specify do
  expect { subject }.to perform_constant_number_of_queries
end
```

Availables modifiers:

```ruby
# You can specify the RegExp to filter queries.
# By default, it only considers SELECT queries.
expect { ... }.to perform_constant_number_of_queries.matching(/INSERT/)

# You can also provide custom scale factors
expect { ... }.to perform_constant_number_of_queries.with_scale_factors(10, 100)
```

### Minitest

TBD


### Configuration

There are some global configuration parameters (and their corresponding defaults):

```ruby
# Default scale factors to use.
# We use the smallest possible but representative scale factors by default.
self.default_scale_factors = [2, 3]

# Print performed queries if true in case of failure
self.verbose = false

# Ignore matching queries
self.ignore = /^(BEGIN|COMMIT|SAVEPOINT|RELEASE)/

# ActiveSupport notifications event to track queries.
# We track ActiveRecord event by default,
# but can also track rom-rb events ('sql.rom') as well.
self.event = 'sql.active_record'
```

## How does it work?

Take a look at our [Executor](https://github.com/palkan/test-prof/tree/master/lib/n_plus_one_control/executor.rb) to figure out what's under the hood.

## Development

```sh
# install deps
bundle install

# run tests
bundle exec rake
```

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/palkan/n_plus_one_control.


## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
