---
layout: post
title: Visualizing Essay Submission Times
date: 2014-02-04
---

I've long held the following hypotheses about my academic behavior: (1) that my ability to responsibly submit homework before deadlines decays exponentially as the quarter progresses and (2) that I end up completing homework at ungodly hours as a result.

In Spring 2011, I took a course on the History and Philosophy of Modern Science (HIPS) taught by [Professor James Evans](http://home.uchicago.edu/~jevans/) that allowed for an interesting test for both myself and others. Every week, each student in the course was required to write a 700 word memo due by Thursday 9AM. These were posted on the course's [Blackboard](https://chalk8.uchicago.edu/) forum and available to all those enrolled.

In the below, I experiment with a way to visualize submission times that captures the sequential nature (yes, I'm aware that the use of histograms would be sensible here). I plot submissions on a polar coordinate system in a way that mimicks a clock. Moreover, the points snake outwards based on the amount of time until the deadline. That is, essays submitted closer to the deadline are further from the center. Note that assignments 1 and 10 are omitted because they had unusual deadlines.

A few things jump out:

- Students really do write essays at ungodly hours (I'm assuming they don't just happen to submit them at 4AM for fun).
- There is always a clump of students submitting their essays at the last minute, or after. Are these always the same students? Probably...
- A fair number of people submitted Assignments 2 and 5 well before the deadline. Assignment 5 was due the same week as the midterm, so some students likely chose to get it out of the way ahead of time.
- Given the above, it does seem like the ability to be responsible and finish homework ahead of time dissipates rather quickly.

I must admit that I like how the graphs look (even if I don't think they're particularly useful). If anyone is interested in doing a more rigorous analysis, the code and data can be found [here](https://github.com/ryw90/hips-essays). Let me know if you learn anything interesting!

<img src="https://raw.github.com/ryw90/hips-essays/master/posting-times.png" style="width: 600px;"/>

P.S. For reference, my submission times were all over the place: 2 - 4:08AM, 3 - 8:42AM, 4 - 8:41AM, 5 - 8:24PM, 6 - 12:22AM, 7 - 4:22AM, 8 - 12:57AM, 9 - 4:10AM. Needless to say that Spring 2011 wasn't the most restful quarter for me.
