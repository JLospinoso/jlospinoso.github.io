---
layout: post
title: C++ Crash Course
image: /images/cppcc.png
date: 2019-07-28 06:00
tag: On Writing a Fast-Paced Introduction
categories: [C, C++, programming, developing, software]
---
[1]: https://www.amazon.com/Programming-Language-hardcover-4th/dp/0321958322/ref=sr_1_1?keywords=stroustrup+C%2B%2B&qid=1556464518&s=gateway&sr=8-1

It's hard for me to believe, but [C++ Crash Course](https://ccc.codes) is done. As a "fast-paced, thorough introduction to modern C++ for experienced programmers," it covers C++17, the Standard and Boost Libraries, and several testing frameworks. I spent nearly three years of nights and weekends drafting this book. It's surreal that the project's coming to an end!

In this short post, I reflect on why I wrote it and what the experience was like.

## Why Another C++ Book

Bjarne Stroustrup first designed C++ in 1985 as a generic programming language. Dozens of authors have written high-quality books about this 34-year old language, so you might wonder why the world needs another one.

I have three reasons.

### Reason 1: Modern C++ Is a New Language

C++ has reinvented itself several times over the past decade. The latest revisions in 2011, 2014, and 2017 breathed new life into this widely used--if a bit musty--language. The community refers to these revisions as _modern C++_ because it truly is a new language.

Many high-quality books written prior to these revisions were fantastic for those wishing to learn C++, but modern C++ has obsoleted portions of the material. For the newcomer, it's not clear which portions still apply to modern C++ and which don't, so I wouldn't recommend a newcomer pick up a title written prior to 2011 at the very earliest.

### Reason 2: There's Room for a Better Introductory C++ Book

Many C++ books written in the modern era are truly excellent. But most of these require significant prior knowledge. Some of my favorites include:

* [The C++ Program Language by Bjarne Stroustrup](http://www.stroustrup.com/4th.html)
* [Effective Modern C++: 42 Specific Ways to Improve Your Use of C++11 and C++14 by Scott Myers](http://shop.oreilly.com/product/0636920033707.do)
* [C++ Templates: The Complete Guide by David Vandevoorde et. al](https://www.pearson.com/us/higher-education/program/Vandevoorde-C-Templates-The-Complete-Guide-2nd-Edition/PGM301384.html)
* [C++ Concurrency in Action by Anthony Williams](https://www.manning.com/books/c-plus-plus-concurrency-in-action-second-edition)
* [The C++ Standard Library: A Tutorial and Reference by Nicolai M. Josuttis](http://www.cppstdlib.com/)

When I began learning C++, I tried several popular titles aimed at beginners, but I came away from each attempt unsatisfied. As C++ Crash Course explains in the Introduction:

> There are some introductory C++ texts available, but these often skip over crucial details as they are geared to people totally new to programming. For the experienced programmer, it's not clear where to dive in.

> I prefer to learn about complicated topics deliberately, building concepts from their fundamental elements. C++ has a daunting reputation because its fundamental elements nest so tightly together, making it difficult to build up a complete picture of the language. When I learned C++, I struggled to get my arms around the language, bouncing between books, videos, and exhausted colleagues.

> *I wrote the book I wish I had four years ago.*

I hope that this book helps other experienced programmers to expand their horizons into the world's premier system programming language.

### Reason 3: I Needed a Challenge

As a lifelong programmer, I've partaken in my share of nonfiction technical books. Or, as my spouse would put it, my eyes are bigger than my bookshelf. Since I can remember, I've always loved reading books about a technical subjects. In my opinion, the book-writing process produces a clearer, crisper, and more comprehensive presentation than you'll find in wikis, blog posts, videos, or other formats.

A few years ago, I was a [stir-crazy indenturee](https://warontherocks.com/2018/07/fish-out-of-water-how-the-military-is-an-impossible-place-for-hackers-and-what-to-do-about-it/) languishing on active duty. I started [a blog](https://lospi.net) and doodled some [open-source projects](https://github.com/jlospinoso), but these short-term endeavors lacked coherency and, more importantly, consistent critical feedback from others.

I'd come across a blog post by [Chris Sanders](https://chrissanders.org/2014/02/so-you-want-to-write-infosec-book/) describing his experience writing nonfiction technical books. His assessment, which largely jives with other authors' ([JP Aumasson](https://research.kudelskisecurity.com/2017/10/16/the-making-of-serious-cryptography),
[Bonnie Eisenman](https://medium.com/@brindelle/writing-a-programming-book-faqs-after-writing-learning-react-native-8a5ea8ce04e),
[Tina Seelig](https://medium.com/@tseelig/so-you-want-to-write-a-book-advice-for-prospective-authors-ce8103558f55), and [Ben Watson](http://www.philosophicalgeek.com/2014/11/10/tips-for-writing-a-programming-book/)), is that writing a book requires:

* a strong will,
* patience,
* thick skin,
* and non-pecuniary motivations.

After thousands of hours writing and editing C++ Crash Course over the course of nearly three years, I can attest to his experiences.

It took twice as long to write half of what I proposed. I began writing a projects-based C++ with a short introduction to the language. I'd send chapters off to the editors that I thought were pretty solid only to receive them back dripping in red pixels. Rather than try to write a rushed C++ introduction that didn't really serve beginners or experienced C++ programmers well, I decided to focus on writing the best introductory modern C++ text I could. It took almost 800 pages. (So the projects-based chapters will have to wait for another book!)

My experience with various expositive formats like scholarly articles, blog posts, and theses, has taught me that writing is hard--but writing a book was even harder. C++ Crash Course appeals to a more general audience. In my experience, the peer review process is more focused on scientific rigor than on clarity, flow, or comprehensiveness.

I can confidently say that, through the brilliant editors at No Starch Press, I produced the best writing of my life. I'm glad I did it, even if I comically underestimated the required effort.

### Farewell
Sending my final approvals to my production editor conjured mixed feelings.

The book was at times my fidget cube, ready to be edited during a soul-crushing teleconference. Other times it was my sleep aid, waiting for me to inject some levity into a particularly dry passage during a bout of insomnia. Always it was a stern tutor, suggesting humility as I struggled to present content I hadn't yet mastered. No matter the time of day or what life events transpired, this book was always there for me. And I'll miss it deeply.

But I'm also beyond elated that it's getting published. I won't say that it's finished--I could have spent a lifetime improving it--but I believe it will be a useful tool to others hoping to build a strong C++ foundation.

Share and enjoy!
