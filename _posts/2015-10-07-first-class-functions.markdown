---
layout: post
title:  "First-Class Functions"
date:   2015-10-07 07:07:23
categories: blog
---

Using Go turned out to be a great choice for this project. It's fast and has some modern features that make it easier for me to do exactly what I want. One of these features is first-class functions.

A function is a specific set of instructions for a computer to execute. Without them, I would have to rewrite the same code over and over again. They can also take arguments and return specific values so that they can be used to, for example, make a calculation. Here's a typical function:

    func stringToInt(str string) int {
        i, err := strconv.Atoi(str)
        if err != nil {
            return -1
        }
        return i
    }

This one takes a string (from the CSV data) and converts it to an integer so calculations can be done. If there is an error (`err != nil`), it returns `-1`, which is a special value in my code that means "missing data". Otherwise (if it hasn't returned already) it returns the value that it calculated.

Here's another function:

    func filterNameSearch(s string) FilterFunc {
        return func(c *College) bool {
            return strings.Contains(c.nameLowercase, s)
        }
    }

This one's a little more complicated, because it returns another function. What? A function returning another function? Yep. That's the real power of first-class functions. `filterNameSearch` actually creates a `FilterFunc` out of your search query. A `FilterFunc` is a special kind of function I defined that takes a pointer to a college as a parameter (`*College`) and returns either true or false (`bool`) depending on whether that college should stay in. The filter can be run on every college by just looping through them and making a list of the ones that the filter returns true on.

Here's an even more complicated function. It creates a filter out of other filters:

    func filterAnd(funcs ...FilterFunc) FilterFunc {
        return func(c *College) bool {
            for _, f := range funcs {
                if !f(c) {
                    return false
                }
            }
            return true
        }
    }

`filterAnd` returns a filter that runs every other filter (in `funcs`) on the college and only returns true if all of the filters return true. This way, filters can be strung together like this:

    filter := filterAnd(
        filterNameSearch(query),
        filterOr(filterPredominantDegree(2), filterPredominantDegree(3)))

The `filterAnd` will make sure that the colleges it returns true on both have a name that contains the `query` and have either 2 or 3 as their predominant degree.

The value of `filter` is actually a FilterFunc that can then be run on all colleges.
