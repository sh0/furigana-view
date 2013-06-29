## FuriganaView ##

* Copyright (C) 2013 sh0 - sh0 (ät) yutani (dot) ee
* [Licensed under Creative Commons BY-SA 3.0](http://creativecommons.org/licenses/by-sa/3.0/)

## General ##

FuriganaView is a widget for Android. It renders text simlarily to TextView, but adds small lines of furigana on top of Japanese kanji. The furigana has to be supplied to the widget within the text.

Demo: ![furigana-view](http://yutani.ee/furigana-view/demo.png)

## Techical ##

### FuriganaView class ###

    TextPaint tp = some_text_view.getPaint();
    String text = "{彼女;かのじょ}は{寒気;さむけ}を{防;ふせ}ぐために{厚;あつ}いコートを{着;き}ていた。";
    int mark_s = 11; // highlight 厚い in text (characters 11-13)
    int mark_e = 13;
    
    FuriganaView m_furigana = (FuriganaView) view.findViewById(R.id.furigana);
    m_furigana.text_set(tp, text, mark_s, mark_e);


### QuadraticOptimizer ###

The placement of multiple furigana labels next to each other is a complicated problem. The first label must be inside the FuriganaView box, every consecutive label must not overlap and the last label must also not clip the FuriganaView rendering box. So for n labels there are n+1 constraints. To give every label equal priority of being as close to the kanji as possible, the cost function of offseting a label from kanji should be quadratic. Therefore a quatratic optimizer should be used to get proper label coordinates.

Optimization problem:

* minimize f(x) = sum x[i]^2 over i
* obey constraints Ax > b

Current solver is absolutly minimal and is not what should be really used. No convergence is checked at any point and in some odd cases the results are incorrect (e.g. some furigana might overlap a bit).

Solver steps:

* Use penalty method to turn constrained problem into unconstrained optimization. New cost function is phi(x).
* Use Newton optimizer for solving unconstrained problem. This involves taking first and second derivatives of phi(x) using phi\_d1(x) and phi\_d2(x) respectively. Also an inverse of the Hessian matrix needs to be calculated (or equivalently solving a linear equations problem).
* Gauss-Seidel method is used for solving the linear system of equations for the Newton optimizer.

This solver pretty much worked on the first go so it has not seen any serious debugging (some commented matrix printing lines still exist in code). The matrices are usually very sparse and with low dimensions so it tends to work pretty well on low width displays.

Ways to improve current solution:

* Check for convergence in all solver steps
* Use a better solver for the Hessian inverse than Gauss-Seidel method (like Cholesky factorization)
* Go from float to double (unlikely to help much)

Proper quadratic problem solvers are difficult to implement. A good and compact solver would be great, but is not on schedule to be implemented right now.
