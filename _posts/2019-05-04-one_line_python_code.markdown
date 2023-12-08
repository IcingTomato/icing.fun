---
layout: post
title:  "What can One-Line-Python-Code do?"
tags: programme python
---

Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.

<iframe src="https://console.python.org/python-dot-org-console/" style="width: 100%; height: 300px;"></iframe>

We can use [Python Online Console](https://console.python.org/python-dot-org-console/) to run the demo.

![python_dot_org_console](http://icing.fun/img/post/2019/05/04/python_dot_org_console.jpg)

# Heart

```python
print('\n'.join([''.join([('ILOVEYOU'[(x-y) % len('ILOVEYOU')] if ((x*0.05)**2+(y*0.1)**2-1)**3-(x*0.05)**2*(y*0.1)**3 <= 0 else' ') for x in range(-30, 30)]) for y in range(30, -30, -1)]))
```

![heart](http://icing.fun/img/post/2019/05/04/heart.jpg)

# Eight Queens

```python
[__import__('sys').stdout.write('\n'.join('.' * i + 'Q' + '.' * (8-i-1) for i in vec) + "\n========\n") for vec in __import__('itertools').permutations(range(8)) if 8 == len(set(vec[i]+i for i in range(8))) == len(set(vec[i]-i for i in range(8)))]
```

![eight_queens](http://icing.fun/img/post/2019/05/04/eight_queens.jpg)

# Fibonacci

```python
print([x[0] for x in [(a[i][0], a.append([a[i][1], a[i][0]+a[i][1]])) for a in ([[1, 1]], ) for i in range(30)]])
```

![fibonacci](http://icing.fun/img/post/2019/05/04/fibonacci.jpg)

