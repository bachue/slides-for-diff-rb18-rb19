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

## Syntax Check

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

# Thanks

---