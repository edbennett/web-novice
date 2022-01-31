---
title: "dicts"
teaching: 12
exercises: 8
questions:
- "What is a Python `dict`?"
- "How do I use a `dict`?"
objectives:
- "Understand what a `dict` is."
- "Be able to create, modify, and use `dict`s in Python."
keypoints:
- "A `dict` is a collection of key-value pairs."
- "Create a `dict` with the syntax `{key1: value1, key2: value2, ...}`."
- "Get and set elements of a `dict` with square brackets: `my_dict[key1] = new_value1`."
---

In the previous episode we saw that some APIs will return data formatted as
JSON, including names (or _keys_) and values associated with them.

Since we would ultimately like to work with data from these APIs in Python, it
would be nice if Python had a data structure that behaved similarly. In the
Software Carpentry [introduction to Python][python-novice-inflammation], we
learned about lists, which are ordered collections of things, indexed by their
position in the ordering. What we would like here is similarly a collection, but
rather than having ordering and indexing by position, instead we would like
elements to have an arbitrary index of our choice.

In fact, Python has such a collection built into it; it is called a `dict`
(short for dictionary). Let's construct one now, to hold [data from the Mayo
Clinic][mayo-caffeine] about caffeine levels in various beverages.

~~~
caffeine_mg_per_serving = {'coffee': 96, 'tea': 47, 'cola': 24, 'energy drink': 29}
~~~
{: .language-python}

We see here that the `dict` is created within curly braces `{}`, and contains
_keys_ and corresponding _values_ separated by a `:`, with successive pairs
being separated by a `,` like in a list.

Again, similarly to a list, we can access elements of the `dict` with square
brackets `[]`. For example, to get the number of mg of caffeine per serving of
coffee, we could use the following:

~~~
print("Coffee has", caffeine_mg_per_serving['coffee'], "mg of caffeine per serving")
~~~
{: .language-python}

~~~
Coffee has 96 mg of caffeine per serving
~~~
{: .output}

We can also replace elements in the same way that we can for a list. For
instance, you may have spotted that the value for `'cola'` is incorrect. Let's
fix that now.

~~~
caffeine_mg_per_serving['cola'] = 22
print(caffeine_mg_per_serving)
~~~
{: .language-python}

~~~
{'coffee': 96, 'tea': 47, 'cola': 22, 'energy drink': 29}
~~~
{: .output}

One thing that we can't do for lists is create new elements by indexing with
`[]`. But `dict`s let us do that, as well:

~~~
caffeine_mg_per_serving['green tea'] = 28
print(caffeine_mg_per_serving)
~~~
{: .language-python}

~~~
{'coffee': 96, 'tea': 47, 'cola': 22, 'energy drink': 29, 'green tea': 28'}
~~~
{: .output}

> ## Ordering
>
> Python `dict`s historically were not ordered&mdash;you would not be guaranteed
> to get back results in the same order that you put them in. In more recent
> versions of Python, `dict`s do preserve the ordering in which they are
> created, so `'green tea'`, having been added most recently, appears at the 
> end.
{: .callout}


## Missing values

`dict`s will throw an error, though, 
if we try to _access_ values for keys that we have not added previously.
~~~
print(caffeine_mg_per_serving['guarana'])
~~~
{: .language-python}

~~~
---------------------------------------------------------------------------
KeyError                                  Traceback (most recent call last)
<ipython-input-2-dff37d2ef7d1> in <module>
----> 1 caffeine_mg_per_serving['guarana']

KeyError: 'guarana'
~~~
{: .output}

To write more robust code, we might like to check
whether we can use a particular key
before trying to access it.
In a list, this is
simple, as we can check whether a particular index is less than the length of
the list. With a dict, we need to use a keyword to check whether a particular
key is `in` the list:

~~~
'coffee' in caffeine_mg_per_serving
~~~
{: .language-python}

~~~
True
~~~
{: .language-python}

Alternatively, if we want to get an element of the list and use a default value
if the key isn't found, we can use the `.get()` method:

~~~
print(caffeine_mg_per_serving.get("coffee", 0))
print(caffeine_mg_per_serving.get("hot chocolate", 0))
~~~
{: .language-python}

~~~
96
0
~~~
{: .output}

(If you don't specify the default value, then Python uses `None` for keys that
are not found.)

## Looping 

Now, a particularly useful thing to do with a list is to loop over it. What
happens when we loop over a `dict`?

~~~
for item in caffeine_mg_per_serving:
    print(item)
~~~
{: .language-python}

~~~
coffee
tea
cola
energy drink
green tea
~~~
{: .output}

Looping (or otherwise iterating) over a `dict` in fact loops over its keys. This
matches with what the `in` keyword does&mdash;it would be strange for the two to
look at different aspects of the `dict`. But sometimes we may want to use the
values as well as the keys in a loop. We could index back into the `dict` via
the key, but that is repetitive. We can instead use the `.items()` method of the
`dict`:

~~~
for drink, quantity in caffeine_mg_per_serving.items():
    print(drink.capitalize(), "contains", quantity, "mg of caffeine per serving")
~~~
{: .language-python}

~~~
Coffee contains 96 mg of caffeine per serving
Tea contains 47 mg of caffeine per serving
Cola contains 22 mg of caffeine per serving
Energy drink contains 29 mg of caffeine per serving
Green tea contains 28 mg of caffeine per serving
~~~
{: .output}

> ## What's in a key?
>
> In this episode, we have used strings as keys, as this is what we're most
> likely to see when working with JSON. This is not a Python restriction,
> however. We can use any "hashable" type as a `dict` __key__; this includes
> strings, numbers, and tuples, among other __immutable__ types.
> Most notably, this excludes lists and dicts (which are mutable).
{: .callout}

> ## `dict`s of functions
>
> What will the following code do?
>
> ~~~
> import numpy as np
>
> operations = {
>     'min': np.min,
>     'max': np.max
> }
>
> def process(array, operation):
>     return operations[operation](array)
>
> print(process([1, 4, 7, 2, -3], 'min'))
> ~~~
> {: .language-python}
>
> When might this kind of behaviour be useful?
>
> Try adjusting the example so that `'mean'` and `'std'` also work as you might
> expect.
>
>> ## Solution
>>
>> This will pull the described function out of the dictionary. This could be
>> useful when you want to allow the user to decide what functionality is desired
>> at run-time, perhaps in a configuration file. Perhaps a choice of inversion
>> algorithms or fitting functions could be offered.
>>
>> To add other functions, the `operations` `dict` could be adjusted as:
>>
>> ~~~
>> operations = {
>>     'min': np.min,
>>     'max': np.max,
>>     'mean': np.mean,
>>     'std': np.std
>> }
>> ~~~
>> {: .language-python}
> {: .solution}
{: .challenge}

## Nested dicts

It is worth noting that the values in a `dict` can be of any type
(this is not true for the keys).
One notable case, is that values can be themselves `dict`s:
~~~
nutrition_values = {'energy': {'units': 'kCal/100g',
                               'values': {'white bread': 273,
                                          'almonds': 512}},
                    'caffeine': {'units': 'mg per serving',
                                 'values': caffeine_mg_per_serving}}
~~~
{: .language-python}
It is then possible to access data using multiple square bracket expressions:
~~~
print("Caffeine content of coffee:", nutrition_values['caffeine']['values']['coffee'])
print("Units:", nutrition_values['caffeine']['units'])
~~~
{: .language-python}
~~~
Caffeine content of coffee: 96 
Units: mg per serving
~~~
{: .output}



[mayo-caffeine]: https://www.mayoclinic.org/healthy-lifestyle/nutrition-and-healthy-eating/in-depth/caffeine/art-20049372
[python-novice-inflammation]: https://swcarpentry.github.io/python-novice-inflammation
