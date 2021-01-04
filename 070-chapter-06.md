### Chapter 6 - Active Record Query Interface

## Introduction

The Ruby on Rails guide [Active Record Query Interface](https://guides.rubyonrails.org/active_record_querying.html) is an excellent resource of the available methods for writing Ruby code to retrieve and modify data in a database.

I won't try to reproduce what the Query Interface guide authors have already written, but in an effort to provide a comprehensive guide, I will include a few examples and possible optimizations in this chapter.

In the previous chapter we discussed the Arel library. The Active Record Query Interface methods use Arel methods under the hood to generate the necessary SQL to access the data in the database.

## Examples

Let's say we have two models, a Chair model and a Leg model. We'll define the relationship between the models as a chair can have many legs.

## Resources

* https://guides.rubyonrails.org/active_record_querying.html

## Wrap-up
