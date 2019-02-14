---
layout: post
title: "Cloju-re-alization"
date: 2019-02-13 20:22:19 +0530
comments: true
categories: clojure functional-programming
---

This is a story of how learning clojure made me realize one of its selling points through subconscious thinking.

I had tried to learn clojure few times in the past and had dropped it because of the prefix notation for expressions and parentheses black-hole. Writing `(def y (+ (* m x) c)` felt very weird after expressing it as `y = m * x + c` during many years of education. I wasn't alone, many of my colleagues had similar feeling about clojure and pure functional programming languages.

<!-- More -->

Some of us attended a training by [clojure evangelist and expert](https://www.linkedin.com/in/rjaju/) who recommended to keep an open mind in the beginning and learn the concepts by solving few basic problems using clojure. As part of this exercise, we wrote a function to find `factors` of a number which made me curious about extending it to find `prime-factors` of a number. My mind automatically started thinking of imperative programming approach and translating it to functional style in clojure.

After few minutes of dabbling, it seemed like a very hard(close to impossible) problem to solve in clojure. It made a dent in my confidence and I needed a fix. I decided to write it in imperative style first and quickly came up with below python program to regain part of my confidence :)

```python
def get_prime_factors(n):
    prime_factors = []
    for i in range(2, n+1):
        if(n % i == 0):
            prime_factors += [i] + get_prime_factors(n/i)
            break
    return prime_factors
# not the most optimal solution, but proved I can code function to find prime factors :)
```

As it was close to end of the day(& week), my mind dropped this problem there. But my subconscious mind hadn't let go of this problem and started giving me this hint at the end of a good night sleep. I needed to breakdown the problem into smaller abstractions i.e prime factors of number is smallest prime factor of the number and prime factors of the quotient. This is the code I wrote in clojure after this sudden flash of thoughts

```clojure
(defn divisible? [n, x]
  (zero? (rem n x))
  )

(def find-first (comp first filter))

(defn smallest-prime-factor [n]
  (find-first #(divisible? n %1) (range 2, (+ n 1)))
  )

(defn prime-factors [n]
  (if (<= n 1)
    ()
    (let [smallest-prime-factor (smallest-prime-factor n)]
      (concat [smallest-prime-factor] (prime-factors (/ n smallest-prime-factor)) )
      )
    )
  )
```

This made me realise, the imperative code I had written earlier was more complex to digest the concept of prime factors when compared to the clojure function. If I have to write this function again in any other language, I would definitely break it down to abstractions defined in the clojure function above. In a way, clojure was making it harder to write bad code!

Well, this was my #cloju-re-alization. What is yours? Leave a comment below or write your own blog post and share