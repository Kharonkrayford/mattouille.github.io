---
title: "Pointers in Go"
layout: post
date: 2017-06-25 16:32
headerImage: false
tag:
- go-lang
- pointers
- software development
star: true
category: blog
author: mattouille
description: A simple explanation of pointers in Go Lang
---

I've been learning Go recently and I've been having some issues solidifying my understanding of pointers, so I thought I'd write a quick blog posts explaining what I've learned and how to easily understand it.

## What are pointers and why are they useful?

Simply put, pointers are variables that store a memory address to another variable. They're useful because they allow us to essentially dynamically allocate memory. Here's a nice anecdote to get you thinking:

Say that we're a mechanic looking for a car that we want to do work on in a very large car lot. To make it more impactful, we have some simple changes we want to make (like changing a light bulb). There is one attendant that can help us in one of two ways:

### The first way
You can tell the attendant the license plate of the car you're looking for. The attendant then goes to a map, locates the car, starts the car, and brings it to your bay. This sounds quite convenient, however, it's not very efficient. It took time for the attendant to find the car, to bring you the car, and now it's sitting in your bay amongst all your tools that you won't use on modifying this car. Even moreso, because the attendant is the only one that can drive the car they have to take it out of your bay as well. This made things quite cumbersome.

### The second way (pointers)

The attendant gives me a copy of the map, highlights where the car is, I go to the car with my lean toolset, change the light and walk back to my bay where I have a whole different car that I'm possibly doing much more work on.

The key here is efficiency. In this case the car is a variable or struct. It's quite expensive to drag a copy (This is how Go handles referencing) through a method, however, it's quite inexpensive to copy the pointer (the map). A memory allocation on a 64 bit machine is 64 bits, that gets especially expensive when you're dealing with a complex structure. The effect is the same, the same data is modified, it's more a issue of how it's referenced. One 64 bit pointer versus a struct which potentially occupies multiple spaces in memory.

## Visualizing my explanation

{% highlight go %}
type Car struct {
  Brand string
  Model string
}

func NewCar(brand, model string) *Car {
  return &Car{
    Brand: brand,
    Model: model,
  }
}
{% endhighlight %}

In this instance, new car is pointing to the struct Car, and in it's return value is accessing the memory allocation for Car and updating variables.

In contrast:

{% highlight go %}
func NewCar(brand, model string) Car {
  return Car{
    Brand: brand,
    Model: model,
  }
}
{% endhighlight %}

In this instance we're bringing the entire struct "Car" into our method. While it's light weight now, as the struct grows this will become quite cumbersome.

## Use of & and *

The best way I can portray this in simple English is this:

`&` gets a memory allocation and allows modifying the struct or variable.
`*` is the variable where the memory reference is stored (pointer) and in our case above, type Car.

Even more simply explained is this code:

{% highlight go %}
package main

import "fmt"

func main() {
	i, j := 42, 2701

	p := &i         // point to i
	fmt.Println(*p) // read i through the pointer
	*p = 21         // set i through the pointer
	fmt.Println(i)  // see the new value of i

	p = &j         // point to j
	*p = *p / 37   // divide j through the pointer
	fmt.Println(j) // see the new value of j
}
{% endhighlight %}

## Conclusion

Hopefully my explanation on pointers in Go has helped you, if you have any suggestions please leave them in the comments below!
