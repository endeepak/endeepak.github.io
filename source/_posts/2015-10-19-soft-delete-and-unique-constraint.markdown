---
layout: post
title: "Soft delete and unique constraint"
date: 2015-10-19 17:59:41 +0530
comments: true
categories: database patterns
---

This post describes a working solution for having unique constraint at DB level for a table with soft deleted rows.

## Problem Context

The system identifies the users by their mobile number. Users are soft deleted by setting `deleted` column value as `1`. A new user can register in with same mobile number as previously deactivated user (The mobile numbers can be recycled by telecoms operators). The unique check at application are susceptible to fail in case of concurrent requests, hence we needed unique constraint at DB to ensure integrity of data. 

<!-- More -->

## The solution should

* work for existing rows imported from legacy system
* work across different databases supported by product

We were able to find different flavors of solutions on net but they were incomplete for our case. They only served as starting point a solution that meets all of our needs mentioned above.

## Final Solution

* Add a new column `deletion_token`
* Add unique constraint for combination `mobile_number, deletion_token`
* A new row added to table would have value of 'NA' `deletion_token`. This is ensured by setting up default value of `NA` at DB level and having constructor of User model(used by ORM) to initialize `deletion_token` to `NA` by default
* Insert a random UUID for soft deleted
* On soft delete of user, generate new UUID and set it on `deletion_token`


| Left align | Right align | Center align |
|:-----------|------------:|:------------:|
| This       |        This |     This     
| column     |      column |    column    
| will       |        will |     will     
| be         |          be |      be      
| left       |       right |    center    
| aligned    |     aligned |   aligned


## Path to the above solution

1. Add unique constraint for columns `mobile_number, deleted`
_Drawback_: This wouldn't allow us to have more than one deleted user with same mobile number

2. Add a unique constraint with a where clause eg: `ADD CONSTRAINT .... WHERE deleted != 1;`
_Drawback_: The where clause in constraint definition is not supported by [all databases](http://stackoverflow.com/a/20962904/69362)

3. Instead of using only 0 or 1 as values for deleted column, increment the number on each delete.
_Drawback_: Expensive as it needs extra db call to retrieve previously soft deleted rows and also expensive to update numbers for existing soft deleted rows in legacy system. It would theoretically fail for concurrent requests without lock.

4. Add a new time-stamp column  called `deleted_at` and add an unique constraint on `mobile_number, deleted_at` _Drawback_: The old rows in legacy system didn't have data for `deleted_at` and populating with dummy data wasn't acceptable.

5. Add a new column  called `deletion_token` and add a constraint on `mobile_number, deletion_token` with NULL value for new rows and UUID for soft deleted rows.
_Drawback_: Few databases don't consider nulls as equal and hence unique constraint does not fail for two rows with same mobile number and NULL value in `deletion_token`

6. Slight modification to point 5, to arrive at the final solution described in the beginning of the post