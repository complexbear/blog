---
title: "Bi-directional Generators"
date: "2018-07-05"
path: "/python_yield_send.md"
labels: "python"
---
In Python most developers are familiar with the `yield` keyword. It is most commonly used in `generator` functions to pass values to the calling code until some stop condition is reached. The advantage this gives is that a sequence of data can be generated on the fly without having to hold the entire data set in memory.

A quick example:
```python

def generator_func():
    values = [1, 2, 3]
    for v in values:
        yield v
    # At this point a StopIteration excepton is raised automatically

gen = generator_func() # Now we have an instance of the generator function

# Get all the values from the generator
while(True):
    try:
        print(next(gen))
    except StopIteration:
        break

# This code outputs
# 1
# 2
# 3
```

The builtin `next` function executes the generator function up until the *next* `yield` statement, at which point the value from the `yield` statement is returned to the calling code. The generator function execution is paused at this point. Note that this means we can have multiple yield statements within a generator function. 

Pretty neat eh? 

**But!** There is more...

In the example above we could pass arguments into our generator function to control or modify the sequence of values that we wish to generate. But once the *instance* of the generator function is created we can't easily alter it's behaviour. 
<small>(yes we could use a closure, but I'm ignoring that, and it would affect all instances of our generator function)</small>

**Unless...** we use the `send` function that all generator functions have. :)

Ooooo, what is this I hear you exclaim! It *sounds* like we can send information back into the generator function. Bingo, that's exactly what it can do.

But why would you need to do that?
Well, perhaps your generator is fetching data out of a database or network card buffer. And the amount of data you wish to fetch in any one call to `next` is dependent upon the system load or some other constraint. In this case we can send information back into the generator that tells it know how much data we wish to be given on the next `next` call. 

I feel an example coming on:
```python
def generator_function(some_resource):
    chunk_size = 5
    index = 0
    while(index < len(some_resource)):
        # The yield statement will pass a slice of the resource back to
        # the calling code, but can also accept data from that code
        # which is received and assigned to the new_chunk_size.
        new_chunk_size = yield some_resource[index : index+chunk_size]
        index += chunk_size
        if new_chunk_size:
            print('updating chunk size to {}'.format(new_chunk_size))
            chunk_size = new_chunk_size

# The resource
data = list(range(50))

# The generator
gen = generator_function(data)

# Let's have a loop that aggressively asks for more data each time
# We have to start with a call to next() because the execution of the
# generator function has not yet reached a yield statement.
chunk = next(gen)
while(chunk):
    try:
        print('size of data received = {0}, data = {1}'.format(len(chunk), chunk))
        new_chunk_size = len(chunk) + 5
        chunk = gen.send(new_chunk_size)
    except StopIteration:
        break

# Output:
# size of data received = 5, data = [0, 1, 2, 3, 4]
# updating chunk size to 10
# size of data received = 10, data = [5, 6, 7, 8, 9, 10, 11, 12, 13, 14]
# updating chunk size to 15
# size of data received = 15, data = [15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29]
# updating chunk size to 20
# size of data received = 20, data = [30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49]
# updating chunk size to 25
```

And, that's kinda it. Useful I think to know that you have this facility available to you, and a nice demonstration of Python's built in message passing between code units.