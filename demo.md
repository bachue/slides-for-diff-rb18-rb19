---
author: Bachue Zhou
title: $ diff rb18 rb19
---

# $ diff rb18 rb19

---

## Our Goal

### OS

  - Debian to Ubuntu 12

### PostgreSQL

  - 8.4 to 9.1

### Ruby

  - 1.8.7 to 2.0

### Rails

  - 2.3.18 to 3.2 (4.0?)

### Other used gems

---

## begin

### &nbsp;&nbsp;`upgrade(:ruby_18, :ruby_19)`

### `rescue`

#### &nbsp;&nbsp;`fix and retry`

### `end`

---

# Syntax Check First

*`find . -name '&#42;.rb' -type f -exec ruby -c {} \; | grep -v 'Syntax OK'`*

---

## Colon in Ruby 1.8

1. case ... when

```ruby
case
when 1: 'a'
when 2: 'b'
when 3: 'c'
end
```

2. if, unless, while, until

```ruby
if a == b: c else d end
```

---

## Colon in Ruby 1.9

1. case ... when

```ruby
case
when 1 then 'a'
when 2 then 'b'
when 3 then 'c'
end
```

2. if, unless, while, until

```ruby
if a == b then c else d end
```

---

## Parentheses in Ruby 1.8

```ruby
[f 1, 2]
```

---

## Parentheses in Ruby 1.9

```ruby
[f(1, 2)]
```

---

## Encoding

* Ruby 1.8: Simple & Easy
* Ruby 1.9: Powerful

---

## Lambda bug in Ruby 1.8

---

# More diffs

---

## YAML Engine

* Ruby 1.8: syck
* Ruby 1.9: psych

---

## Compatibility

* BAD
* [example](http://galeki.is-programmer.com/posts/32636.html)
* Switch ( Not Recommanded ) :

```ruby
begin
  require 'syck'
rescue LoadError
end
require 'yaml'
YAML::ENGINE.yamler = 'syck' if defined?(YAML::ENGINE)
ENV['TEST_SYCK'] = 'true'
```

---

## Require in Ruby

* Ruby 1.8:

```ruby
require 'config/boot'
```

* Ruby 1.9:

```ruby
require_relative 'config/boot'
```

* Compatibility:

```ruby
require File.expand_path(File.dirname(__FILE__) + '/config/boot')
```

---

# Object
## Object#to_s

* Ruby 1.8:

```ruby
[1,2,3,:four,[5,6], {:seven => 8}].to_s
# => "123four56seven8"
```

* Ruby 1.9:

```ruby
[1,2,3,:four,[5,6], {:seven => 8}].to_s
# => "[1, 2, 3, :four, [5, 6], {:seven=>8}]"
```

* Should avoid using this method

---

# Array

---

## Convert to Array

* Ruby 1.8: Object#to_a
* Ruby 1.9: Array()

---

## Array(string)

* Ruby 1.8:

```ruby
Array("21312\n423424\n234234")
# => ["21312\n", "423424\n", "234234"]
```

* Ruby 1.9:

```ruby
Array("21312\n423424\n234234")
# => ["21312\n423424\n234234"]
```

* This is the reason why Rails 2.3 Cookie cannot work in Ruby 1.9

---

## Patch

```ruby
if RUBY_VERSION =~ /^1\.9/
  module Kernel
    def Array_with_rb19_fix(element)
      if !element.nil? && element.is_a?(String)
        element.each_line.to_a
      else
        Array_without_rb19_fix element
      end
    end

    alias_method :Array_without_rb19_fix, :Array
    alias_method :Array, :Array_with_rb19_fix
  end
end
```

---

# Symbol

* Ruby 1.8: Symbol#to_i
* Ruby 1.9: could be like Symbol#object_id

---

# Date

---

## ParseDate

* Ruby 1.8:

```ruby
require 'parsedate'
ParseDate.parsedate "Tuesday, July 5th, 2007, 18:35:20 UTC"
# => [2007, 7, 5, 18, 35, 20, "UTC", 2]
```

* Ruby 1.9:

```ruby
require 'date'
Date._parse(str, comp).
  values_at(:year, :mon, :mday, :hour, :min, :sec, :zone, :wday)
```

---

## Time.now

* Ruby 1.8: Thu Sep 19 22:55:12 +0800 2013
* Ruby 1.9: 2013-09-19 22:55:12 +0800

---

# File

* Ruby 1.8:

```ruby
require 'ftools'
File::copy(src, dst)
File::rename(old, new)
```

* Ruby 1.9:

```ruby
require 'fileutils'
FileUtils.cp(src, dst)
File.rename(old, new)
```

---

##

---

# Thanks

---