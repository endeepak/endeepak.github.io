---
layout: post
title: "Joy of finding a math series"
date: 2017-10-28 12:46:47 +0530
comments: true
categories: maths
---

I was trying to come up with a question for preliminary logic round for interviewing. I decided to form a tricky math series and tried to figure out 2 math series which looks same in the beginning and diverges after a while. Something on the lines of

```
series x => x1, x2, x3, x4, x5
series y => y1, y2, y3, y4, y5

where x1=y1, x2=y2, x3=y3 and x4!=y4, x5!=y5
```

So the question could be

```
Write the missing number in below series
x1, x2, x3, ? , y5
```

<!-- More -->

To figure out these series, my approach was to start with some series x and find a pattern which doesn't hold good after few numbers. After few unsuccessful attempts, tried my luck with series of squares

```
1, 4, 9, 16, 25, ...
```

Voilà! difference of consecutive squares were not only a odd number, they were consecutive odd numbers. Tried it for a long series and found it true for all the squares!

```
1-0, 4-1, 9-4, 16-9, 25-16,... => 1, 3, 5, 7, 9,..
```

From what I knew of squares of numbers, this wasn't very obvious. I was very excited to check if this is already known or I was the first one to discover this :). Simple web search presented ton of articles on this observation.

That was disappointing but reading this [link](http://mathcentral.uregina.ca/QQ/database/QQ.09.99/nghiem1.html) about a mathematician appreciating a student's effort in similar finding was comforting and encouraging to carry on with my quest



