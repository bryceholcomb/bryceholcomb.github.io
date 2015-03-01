---
layout: post
title:  "Querying with ActiveRecord"
date:   2015-02-10 21:35:07
summary: "Managing your relationship with your DB"
---
## History of ActiveRecord
## Public Methods
- #where
- #from
- #select, both types(first: Array#select with block, AREL object)
- #group
- #having
- #joins
- #limit
- #uniq
- #not, new to rails 4
- #unscope
- #none
- #offset
- #order
- #includes, vs. preloading
- #merge
- #references
- #reorder
- #distinct
- #index, and why its expensive
- #scopes are actually class methods, and you can chain them, wtf are lambdas
- #default_scope
- #Finder methods are deprecated

## AREL objects vs. Arrays
### Do not use Arrays

## Object Oriented Querying
Long scope methods violate object oriented design especially if you are using #join. Use #merge instead and call the scope made on the other table

## Adequate Record in Rails

## What this looks like in a Rails app
### Performance
