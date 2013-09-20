---
author: Bachue Zhou
title: $ diff rb18 rb19
---

<style type="text/css">
  pre code { background: transparent; }
</style>

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

```bash
$ find . -name '&#42;.rb' -type f -exec ruby -c {} \; | \
    grep -v 'Syntax OK'
```

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

## Parentheses in Ruby 1.8

```ruby
[f 1, 2] # ambiguous! [f(1), 2] ? or [f(1, 2)] ?
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

## In Rails 2.3

```ruby
content = File.read '/bin/sh'
content.blank?
# => ArgumentError: invalid byte sequence in UTF-8
```

---

<style type="text/css">
  #step-12 pre code { font-size: 60%; line-height: 24px; }
</style>

## PATCH

```ruby
if defined?(Encoding)
  class String
    def blank?
      begin
        self !~ /\S/
      rescue ArgumentError
        if /invalid byte sequence/ =~ $!.message && !@_tried_to_fix_encoding_error_in_ruby_19
          self.force_encoding(Encoding::BINARY)
          @_tried_to_fix_encoding_error_in_ruby_19 = true
          retry
        else
          raise
        end
      end
    end
  end
end
```

> [And Apply this Patch](https://developer.uservoice.com/blog/2012/03/04/how-to-upgrade-a-rails-2-3-app-to-ruby-1-9-3/)

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

* Compatible:

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

# String

---

## String#[]

* Ruby 1.8:

```ruby
'hello'[0]
# => 104
```

* Ruby 1.9:

```ruby
'hello'[0]
# => "h"
```

* Compatible:

```ruby
'hello'[0].chr
# => "h"
```

---

## String#length

* Ruby 1.8:

```ruby
'你好'.length
# => 6
```

* Ruby 1.9:

```ruby
'你好'.length
# => 2
```

* Compatible:

```ruby
'你好'.bytesize
# => 6
```

---

# Array

---

## Convert to Array

* Ruby 1.8: Object#to_a
* Ruby 1.9: Array()

---

## Random choice

* Ruby 1.8:

```ruby
[1, 2, 3, 4].choice
```

* Ruby 1.9:

```ruby
[1, 2, 3, 4].sample
```

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
unless ''.respond_to?(:each)
  module Kernel
    def Array_with_rb19_fix(element)
      if !element.nil? && element.is_a?(String)
        Array_without_rb19_fix(element.each_line)
      else
        Array_without_rb19_fix element
      end
    end

    alias_method_chain :Array, :rb19_fix
  end
end
```

---

# Symbol

* Ruby 1.8: Symbol#to_i
* Ruby 1.9: could be like Symbol#object_id

---

# Hash

---

## Hash#select

* Ruby 1.8:

```ruby
{'a' => 1, 'b' => 2}.select {|x, y| y.odd?}
# => [["a", 1]]
```

* Ruby 1.9:

```ruby
{'a' => 1, 'b' => 2}.select {|x, y| y.odd?}
# => {"a"=>1}
```

---

## Add new keys during interation

* Ruby 1.8:

```ruby
h = {:a => 1, :b => 2}
h.each {|k, v| h[k.to_s[0].succ.chr.to_sym] = v + 1 }
# => {:b=>2, :e=>5, :c=>3, :d=>4, :a=>1}

h = {'a' => 1, 'b' => 2}
h.each {|k, v| h[k[0].succ.chr] = v + 1 }
# => {"c"=>3, "b"=>2, "a"=>1}
```

* Ruby 1.9:

*RuntimeError: can't add a new key into hash during iteration*

---

# Regex

## In ERB

*warning: regexp match /.../n against to UTF-8 string*

---

# Date

---

## Date.new!
## Date.new0
## Date.new1
## Date.new2
## Date.new3

---

```ruby
Date.new(2013, 9, 21).strftime  # => "2013-09-21"
Date.new!(2013, 9, 21).strftime # => "-4707-06-07"
Date.new0(2013, 9, 21).strftime # => "-4707-06-07"
Date.new2(2013, 9, 21).strftime # => "2013-01-09"
Date.new3(2013, 9, 21).strftime # => "2013-09-21"
```

---

## new!, new0, new1: Julian day

> 儒略日的起点订在公元前4713年（天文学上记为 -4712年）1月1日格林威治时间平午（世界时12:00），即JD 0指定为UT时间B.C.4713年1月1日12:00到UC时间B.C.4713年1月2日12:00的24小时。每一天赋予了一个唯一的数字，顺数而下，如：1996年1月1日12:00:00的儒略日是2450084。这个日期是考虑了太阳、月亮的轨道运行周期，以及当时收税的间隔而订出来的。

## new2

```ruby
Date.new2(2013, 256).strftime
# => "2013-09-13" # Programmers' Day!
```

---

## Patch

<style type="text/css">
  #step-36 pre code { font-size: 55%; line-height: 20px; }
</style>

```ruby
require 'date'
require 'tzinfo'

unless DateTime.respond_to?(:new!) || DateTime.respond_to?(:new0)
  module TZInfo
    module RubyCoreSupport
      HALF_DAYS_IN_DAY = rational_new!(1, 2)

      def self.datetime_new!(ajd = 0, of = 0, sg = Date::ITALY)
        jd = ajd + of + HALF_DAYS_IN_DAY

        jd_i = jd.to_i
        jd_i -= 1 if jd < 0
        hours = (jd - jd_i) * 24
        hours_i = hours.to_i
        minutes = (hours - hours_i) * 60
        minutes_i = minutes.to_i
        seconds = (minutes - minutes_i) * 60

        DateTime.jd(jd_i, hours_i, minutes_i, seconds, of, sg)
      end
    end
  end
end
```

> [Copy from: How to upgrade a Rails 2.3 app to Ruby 1.9.3](https://developer.uservoice.com/blog/2012/03/04/how-to-upgrade-a-rails-2-3-app-to-ruby-1-9-3/)

---

## Date.exist?

* Ruby 1.8:

  - Date.exist?
  - Date.exist1?
  - Date.exist2?
  - Date.exist3?

* Ruby 1.9:

  - Date.valid_date?
  - Date.valid_jd?
  - Date.valid_original?

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

## BigDecimal

```ruby
require 'bigdecimal'
require 'bigdecimal/util'
99.99.to_d.to_s
```

* Ruby 1.8:

```ruby
# => '0.9999E2'
```

* Ruby 1.9:

```ruby
# => '0.9998999999999999E2'
```

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

## No RDoc::Usage any more

---

# Thanks

---