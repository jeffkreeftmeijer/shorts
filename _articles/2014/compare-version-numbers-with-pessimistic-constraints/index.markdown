To compare version numbers using [pessimistic constraints]() (using `~>`) in Ruby, you could use [Rubygems' `Gem::Version`](http://stackoverflow.com/a/3064161/325603) or [do hacky things with `Gem::Dependency`](http://stackoverflow.com/a/17449664/325603). 

Instead, I decided to write [a simple and understandable `Version` class](https://gist.github.com/jeffkreeftmeijer/912f3f9fd72b8ecd841c) that does everything I need:

``` ruby
require 'version'

Version.new('1').matches?('~>', '1') # => true
Version.new('1.2').matches?('~>', '1.0') # => true
Version.new('1.2.3').matches?('~>', '1.0.0') # => false
```

Using [Ruby's `Comparable` module](http://www.ruby-doc.org/core-2.1.3/Comparable.html) and a method that figures out the next incompatible version, implementing the `=`, `>`, `<` and `~>` operators could be done in [40 lines of Ruby](https://gist.github.com/jeffkreeftmeijer/912f3f9fd72b8ecd841c):

``` ruby
class Version
  include Comparable
  attr_reader :major, :minor, :patch
 
  def initialize(number)
    @major, @minor, @patch = number.split('.').map(&:to_i)
  end

  def to_a
    [major, minor, patch].compact
  end

  def <=>(version)
    (major.to_i <=> version.major.to_i).nonzero? || 
    (minor.to_i <=> version.minor.to_i).nonzero? ||
    patch.to_i <=> version.patch.to_i
  end

  def matches?(operator, number)
    version = Version.new(number)
    self == version

    return self == version if operator == '='
    return self > version  if operator == '>'
    return self < version  if operator == '<'
    return version <= self && version.next > self if operator  == '~>'
  end

  def next
    next_splits = to_a

    if next_splits.length == 1
      next_splits[0] += 1
    else
      next_splits[-2] += 1
      next_splits[-1] = 0
    end

    Version.new(next_splits.join('.'))
  end
end
```
