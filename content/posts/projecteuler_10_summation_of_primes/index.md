+++
title = 'Project Euler 10: Summation of Primes'
date = 2024-05-01T19:21:54-05:00
tags = ['programming', 'math', 'euler100']
+++


# Project Euler

This is problem 10 of the [Project Euler](projecteuler.net) archive titled **Summation of Primes**.
I figured that publishing every problem in the top 100 was not feasible as the point is to solve problems rather than write about them.
However, I believe it's good to publish ones I find interesting from a problem solving or mathematical perspective.
The interesting part of this problem was how I thought it should be solved, leading me to create something that can efficiently check if a number is prime.
Again, as per the project's rules, I'm only allowed to publish up to the first 100 listed problems in the archive.
This earns these posts the tag of **euler100**.

# Problem

>The sum of the primes below $10$ is $2 + 3 + 5 + 7 = 17$.
Find the sum of all the primes below two million.

I typically try and brute force as much as possible.
This way, I learn the limitations of my approach, which will help me later on.
To solve this I can simply try and perform a modulo operation on all numbers less than the candidate prime.
Then, collect all my primes and sum.
Below is my initial brute force approach.



```python
def is_prime_brute(candidate):

    n = 2
    if candidate==1:
        return False
    if candidate==2:
        return True

    while n < candidate:
        if candidate % n == 0:
            return False
        n+=1
        
    return True
```


```python
prime_sum = 2
bound = 2_000_000

# we skip every even number since we know it's not prime
for n in range(1, bound+1, 2):
    if is_prime_brute(n):
        prime_sum += n

```



First I can show that this works with the given example in the problem statement.


```python
prime_sum = 2
bound = 10

# we skip every even number since we know it's not prime
for n in range(1, bound+1, 2):
    if is_prime_brute(n):
        prime_sum += n
print(prime_sum)
```
    17
    

However, I did not use this to find the sum for primes under 2 million because of how long it took to check up to a quarter of the way there (about 4 minutes).
I could just wait it out but, there's a small anecdote on the website that is meant to save you some computation time and challenge you to think.

> **I've written my program but should it take days to get to the answer?**
>
> Absolutely not! Each problem has been designed according to a "one-minute rule", which means that although it may take several hours to design a successful algorithm with more difficult problems, an efficient implementation will allow a solution to be obtained on a modestly powered computer in less than one minute.

I decided to take this to heart and attempt to design something more clever.

# Brute Force Issues

I've already implemented one optimization above: only check odd numbers.
I also just return `n=1` and `n=2` as adhoc facts to allow a blanket elegance to my `while` loop.
This was very simple to add with a few characters to the `range` function.
Unfortunately, this doesn't really optimize it by much.
Most of the time in the function will be when we encounter a prime number.

To demonstrate this, I've recorded it with the **line magic** `timeit` function for prime numbers up to $10,000$.




```python
%%timeit

prime_sum = 2
bound = 10000

prime_list = []
# we skip every even number since we know it's not prime
for n in range(1, bound+1, 2):
    if is_prime_brute(n):
        prime_list.append(n)
        prime_sum += n

```

    268 ms ± 2.19 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
    

```python
%%timeit
for n in prime_list:
    is_prime_brute(n)
```

    268 ms ± 2.01 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
    


```python
%%timeit
bound = 10000
for n in range(1, bound+1, 2):
    if n in prime_list:
        continue
    is_prime_brute(n)
```

    30.3 ms ± 199 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)
    

The first run is creating this list of primes up to $10,000$.
The second simply applies the `is_prime_brute()` function call to the primes that were found.
The last run applies the function to all other numbers that aren't primes.

With some slight fluctuations in the times (and even `timeit` averages not shown), just checking primes is the majority of our computation time. 

The reason is simple, we have to iterate all the way up to that prime number before we're absolutely sure this is a prime.
Or at least that's the way the function is currently written.

I did not let this code execute fully this way because of how long it took.
In complexity, this takes $O(n^2)$ time. 
Not including whatever the modulo operator needs to work.

# Optimizing Prime Validation

In solving this problem I tried to think of some easy facts about integer division.
The first thing I realized was that, no integer is soundly divisible by any number greater than it's half.
Or in math terms

$$ a  \left(\mod (\frac{a}{2} + 1) \right) \ne 0$$

So that means I can make my upper bound to the loop for division 

`divisor <= candidate//2`

But if it's not divisible by $2$, then the next possible divisor would be $3$.
However, there are no integers between $2$ and $3$, so that means there are no divisors between $a/3$ and $a/2$. 
We can basically skip a whole set of numbers and clamp the termination of the loop downwards as we test.
So, on the next iteration, our termination criteria is now 

`divisor <= candidate//3`

And so on and so forth, better written as 

`divisor <= candidate//divisor`

I'll optimize further and skip all even numbers in the loop beyond $2$.
Since this loop takes the longest on the primes, skipping even numbers practically cuts the computation time in half.

This implementation is below.


```python
# My solution

def is_prime(candidate):

    if candidate ==1:
        return False
    if candidate % 2 == 0:
        return True

    divisor = 3

    # lower the termination bound to limit our checks to realistic numbers.
    while divisor <= (candidate)//divisor:
        if candidate % divisor == 0:
            return False
        divisor+=2
        
    return True
    
```


```python
prime_sum = 2
bound = 2_000_000
# we skip every even number since we know it's not prime
for n in range(1,bound+1,2):
    if is_prime(n):
        prime_sum += n
print(prime_sum)
```

    142913828922
    
This takes the generation of the solution from at least greater than 4 minutes in our brute force approach, to just **5 seconds**.

The time complexity here was not necessarily reduced by some clever generating function such as [triangular numbers](https://en.wikipedia.org/wiki/Triangular_number) for counting (in the first project euler problem).
Instead, the bound for the number decreases as a function of (1/n) and cuts out a lot of extra number checks that don't need to happen.
In fact, there are definitely better ways of doing this but I stumbled upon it myself, because I was trying to use prior knowledge to solve it. 
And usually when you throw your current skill set against a harder problems, you will have to change approach solve it.

I did forget about a possibly easier method that is somewhat similar to the mindset of my approach: [Sieve of Eratosthenes](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes).
The idea being that we create a list, and poke out numbers that are divisors of larger numbers.
The similarity here is, checking a lot of numbers, but also skipping numbers you are sure aren't primes to shorten computation.
The difference is that the sieve is much more efficient for this problem.



# Sieve of Eratosthenes

Credit to **lassevk** on the Project Euler forums for this solution. 
I've modified it a bit to make it easier to read and explain.
Before proceeding, I would read about the [Sieve of Eratosthenes](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes).
It's very easy to understand given the right visuals.

|![](./feature.gif)|
|:--:|
|*source: https://en.wikipedia.org/wiki/File:Animation_Sieve_of_Eratosth.gif*|

Below is the modified implementation of the algorithm.


```python
# Seive of Eratosthenes

bound = 2000000

excluded_mask = [1] *bound 
current_number = 2
prime_sum = 0

while current_number < bound:

    if excluded_mask[current_number] == 1:

        prime_sum += current_number
        sieve_pointer = current_number

        while sieve_pointer < bound:

            excluded_mask[sieve_pointer] = 0
            sieve_pointer += current_number

    current_number += 1


print(prime_sum)
```

    142913828922
    

The point here is to you have a list of numbers, and you also have a some sort of way to track which ones are left in the list and which ones are not.
We crawl through the list from first to last multiple times, but only if the number is marked as *still there*.
This way of tracking something in a binary way is sometimes referred to as a **bit mask**.
Every time we crawl the list, the first element we see is considered a prime, and then removed from the list.
My mask uses a value of `1` to represent that it still exists within the list.
In step form:
1. Check if this number is in the list
2. If it is, it is the next prime and can be added to the sum, go to step 4
3. If not, go to the next number and back to step 1
4. Set a pointer (index tracking variable) at the current number (we are tracking two numbers, the original number that we just added to our sum, and the number we are going to remove from the list)
5. Remove the number from the list.
6. Increment the number by the first number we track, the prime number
7. Remove every number that is a multiple of it.
8. Increase increase the counter and go back to step 1.

This works better than my solution, because we're not checking if every number is a prime, we're just generating them on the fly which is easier.


# Final Thoughts

I liked my solution because I thought it was very clever and I'd like to keep it or see some use of it while I continue to solve these problems.
The difference between the sieve and my solution is that I can quickly check if any number is a prime, while the sieve generates all primes up to that point.


Because of this, I'd like to know which is faster for simply checking if one number is prime.
Using the `timeit` function I can compare the two methods below.
I will need to modify the sieve algorithm slightly to check if the number is prime


```python
def check_prime_from_sieve(bound):
    marked = [1] *bound 
    current_number = 2
    prime_sum = 0
    prime_list = []
    while current_number < bound:
        if marked[current_number] == 1:
            prime_list.append(current_number)
            prime_sum += current_number
            sieve_pointer = current_number
            while sieve_pointer < bound:
                marked[sieve_pointer] = 0
                sieve_pointer += current_number
        current_number += 1
    
    if prime_list[-1]==bound:
        return True
    else:
        return False


candidate_non_prime = 321817
candidate_prime = 1999993    
```

For a number I know **is not prime** the times are:



```python
%%timeit
is_prime(candidate_non_prime)
```

    15.5 µs ± 128 ns per loop (mean ± std. dev. of 7 runs, 100,000 loops each)


```python
%%timeit
check_prime_from_sieve(candidate_non_prime)
```

    42.5 ms ± 178 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)


And for a number I know **is prime** the times are:


```python
%%timeit
is_prime(candidate_prime)
```

    42.3 µs ± 274 ns per loop (mean ± std. dev. of 7 runs, 10,000 loops each)


```python
%%timeit
check_prime_from_sieve(candidate_prime)
```

    298 ms ± 11.2 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

The reasoning for the sieve being slower in checking a single prime, is the same for it being faster when getting a list of primes.
It has to generate the primes before we can use them.
Regardless, I have a couple of new functions to use as I continue to crawl through the problem sets.