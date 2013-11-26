---
layout: post
title: "Ruby, Combinatorics and Burritos"
description: "Just how many possible burrito combinations are there?"
category: posts
tags: [math, programming, clojure]
comments: true
---

<figure>
    <img src="http://blog.imeri.ca/images/burrito.jpg" alt="Image found on http://www.flickr.com/photos/jeffcam/377725212/">
</figure>

<br />

The [Freebirds World Burrito](http://freebirds.com/) off of 620 in North Austin is claiming "Over one billion burrito configurations". You
probably know where this is going.

- Tortillas: (4)
    - Spinach, Flour, Cayenne, Wheat

- Ingredients (25)
    - Cilantro Lime Rice, Spanish Rice, Refried Beans, Whole Pintos, Black Beans, Lettuce, Mixed Greens, White Onions,
    Red Onions, Tomatoes, Pico De Gallo, Roasted Peppers & Onions, Roasted Garlic, Lime Juice, Roasted Limes,
    Ranch Dressing, Ancho Dressing, Tomatillo Dressing, Salsa, Poblano Salsa, BadAss Bbq, Mild Tomatillo, Hot Tomatilla
    Death Sauce, Habanero Sauce


#### Rules

- Our working definition of a burrito is a tortilla with at least one ingredient inside it.
- Order doesn't matter. (tortilla + salsa + chicken) == (chicken + salsa + tortilla)
- No doubling up on Tortillas. What kind of person orders a burrito with an extra layer of tortillas anyway, amiright?
- Size doesn't matter. Let's assume the burrito is the super monster (huge).
- Doubling up on meats or anything additional is fine.


#### Brainstorm

Before we start writing code, let's have a think about the problem. Are we looking for a count of an array of
permutations? Or are we looking for a cartesian product? Or the factorial of N number of ingredients? It helps me to
solve the most extreme simple case: How many combinations would exist if there were only one tortilla and two ingredients?

- Tortilla + ingredient_one
- Tortilla + ingredient_two
- Tortilla + ingredient_one + ingredient_two


#### Implementations
Let's see if we can come up with the right answer using a few methods commonly associated with combinatorial problems.
As of 1.9.2, Ruby's Array class includes a method called `permutation()`. Maybe this will help us:

{% highlight ruby %}

    def possible_combinations(ingredients)
        return (1..ingredients).to_a.permutation.count
    end

    possible_combinations(2)
    => 2
{% endhighlight %}

This is wrong because `Array.permutation()` is returning combinations that must include all values. See below:

{% highlight ruby %}

    def list_combinations(ingredients)
        return (1..ingredients).to_a.permutation.map {|n| n}
    end

    list_combinations(3)
    => [[1, 2, 3], [1, 3, 2], [2, 1, 3], [2, 3, 1], [3, 1, 2], [3, 2, 1]]

{% endhighlight %}

In burrito-land, these are all the same burrito. So `Array.permutation()` is not the way to go. Let's see if we can arrive at
the answer using the factorial value of N number of ingredients.



{% highlight ruby %}

    class Integer
       def factorial
       (1..self).reduce(:*)
       end
    end

    num_of_ingredients = 3
    num_of_ingredients.factorial()
    => 6

{% endhighlight %}


This is also wrong because there is actually seven unique combinations of three ingredients:


{% highlight ruby %}
 ingredients = [:salsa, :chicken, :cheese]
 # All combinations
 [  [:salsa],
    [:chicken],
    [:cheese],
    [:salsa, :chicken],
    [:salsa, :cheese],
    [:chicken, :cheese],
    [:chicken, :cheese, :salsa]  ]


{% endhighlight %}

<figure>
    <center><img src="http://blog.imeri.ca/images/sherlock.gif"></center>
</figure>

 <br />

Let's have a look at `Array.combinations()` to see if we can build subsets of the set including the set itself.


{% highlight ruby %}

    def list_combinations(ingredients)
       ingredients_array = (1..ingredients).to_a
       combinations = 1.upto(ingredients_array.size).flat_map do |n|
            ingredients_array.combination(n).to_a
       end
       return combinations
    end

    list_combinations(3)
    => [[1], [2], [3], [1, 2], [1, 3], [2, 3], [1, 2, 3]]

{% endhighlight %}

We have a winner! All we need to do is count the number of inner arrays.


{% highlight ruby %}

    def list_combinations(ingredients)
       ingredients_array = (1..ingredients).to_a
       combinations = 1.upto(ingredients_array.size).flat_map do |n|
            ingredients_array.combination(n).to_a
       end
       return combinations
    end

    def possible_combinations(ingredients)
        list_combinations(ingredients).count
    end

    burrito_ingredients = 25
    tortillas = 4
    possible_combinations(burrito_ingredients) * tortillas
    => 134217724 # "134,217,724"

{% endhighlight %}

It would appear that for the first time in history, someone in marketing has exaggerated‎ the truth. There are actually
134,217,724 ways to configure a burrito at Freebirds :) .

#### Summmary

Solving combinatorical problems can quickly go in the wrong direction if you don't gain a good footing on the problem
you're trying to solve. As a general heuristic , when ordering or position of elements in your collection matter,
look at using permutations to solve your problem. When you need to find unique subsets within a set, use combinations
or powersets.

Also, check out the gem [combinatorics](https://github.com/postmodern/combinatorics) for solving more complex problems
relating to combinatorics.








