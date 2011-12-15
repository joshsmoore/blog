---
title: Ruby Reduce Alpha Release
layout: post
---

Today I have released Ruby Reduce.  A gem, that as the name implies, is an implementation of the google's [map reduce](http://www.mapreduce.org/) function in Ruby.  Right now the implementation is very limited in several key ways:

* it only accepts Rails 3 logs as input
* output written to MongoDB
* single threaded no distribution.


Despite these limitations it does work and it will allow you to reduce Rails log files so that you can analysis the data that they contain.  More information and usage at the [Github page](https://github.com/joshsmoore/ruby_reduce)
