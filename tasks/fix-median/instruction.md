`/app/stats.py` has a `median(values)` function. It returns the wrong answer for lists with an even number
of items: their median is the mean of the two middle values. Fix it.

It must keep working for lists of odd length and for unsorted input, must not change the list it is given,
and must raise `ValueError` for an empty list.
