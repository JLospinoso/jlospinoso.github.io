---
layout: post
title: Lambda expressions and C++11
image: /images/functional.svg
date: 2015-03-11 07:00
tag: Using lambdas can make for some wonderfully elegant code
categories: [c++, developing, software, visual studio]
---
[1]: http://en.wikipedia.org/wiki/Anonymous_function
[2]: https://www.haskell.org/
[3]: http://fsharp.org/
[4]: http://www.oracle.com/technetwork/java/javase/overview/java8-2100321.html
[5]: https://msdn.microsoft.com/en-us/library/bb397897.aspx
[6]: http://www.pavley.com/tag/functional-programming/
[7]: https://github.com/JLospinoso/LambdasCpp11
[8]: http://en.wikipedia.org/wiki/Imperative_programming
[9]: http://blog.codinghorror.com/code-smells/
[10]: https://msdn.microsoft.com/en-us/library/aa730844%28v=vs.80%29.aspx
[11]: http://en.wikipedia.org/wiki/Map_%28higher-order_function%29
[12]: http://en.cppreference.com/w/cpp/language/lambda
[13]: http://stackoverflow.com/questions/19208561/scala-predicates-and-filter-functions
[14]: http://en.cppreference.com/w/cpp/language/storage_duration
[15]: http://www.meetingcpp.com/index.php/imprint.html
[16]: https://twitter.com/meetingcpp
[17]: http://en.cppreference.com/w/cpp/algorithm/count
[18]: http://en.cppreference.com/w/cpp/algorithm/transform

According to [Wikipedia][1], a lambda expression (*lambda*) is "a function that is not bound to an identifier." In other words, it's a function that you can pass around without having to name (define) it separately. These are ubiquitous in functional programming languages like [Haskell][2] and [F#][3]. Why?

Well, consider the corollary in imperative languages. Consider this snippet:

```cpp
auto result = answer_to_life(6, 7, "What's the question?", true);
```

All of the arguments passed to the function `answer_to_life` are "not bound to an identifier." If we were required to bound all of these parameters to an identifier, we would be stuck with this mess:

```cpp
int number1 = 6, number2 = 7;
char *question = "What's the question?";
bool deepThought = true;
auto result = answer_to_life(number1, number2, question, deepThought);
```

Yuck.

"Well," you might ask, "why should I care? I almost *never* pass functions as parameters."

Maybe you should be.

# Adapting functional patterns
Other top-tier languages like [Java 8][4] and [C# (.NET 3.5)][5] have promoted functions to first-class citizens for good reason. For some problems, it is much more elegant to pass functions around rather than data.

Let's build a helper class that works on lists of words. First, let's take a `vector` of `std::string`s and return a `vector` of their corresponding lengths. Check out this unit test *a la* `Microsoft::VisualStudio::CppUnitTestFramework`:

```cpp
TEST_METHOD(Counts)
{
	WordListHelper helper{};
	std::vector <std::string> input { "o", "tw", "thr" };
	auto lengths = helper.counts(input);

	Assert::AreEqual(1, lengths.at(0));
	Assert::AreEqual(2, lengths.at(1));
	Assert::AreEqual(3, lengths.at(2));
}
```

`<aside>`*Aren't tests a beautiful way of expressing functional requirements?*`</aside>`

We'll frame out our class:

```cpp
#include <vector>
#include <string>
#include <functional>

class WordListHelper
{
public:
	WordListHelper();
	std::vector<int> counts(const std::vector<std::string> words);
}
```

And fill in the method *just so it compiles*:

```cpp
std::vector<int> WordListHelper::counts(const std::vector<std::string> words)
{
	std::vector<int>{ 42 };
}
```

Run the test, make sure it fails, then let's have a hack at an implementation of `WordListHelper::counts`:

```cpp
std::vector<int> WordListHelper::counts(const std::vector<std::string> words)
{
	std::vector<int> out{};
	for each (auto word in words)
	{
		out.push_back(word.length());
	}
	return out;
}
```

This is probably pretty close to what you were thinking: loop through the input, get the length, pack it onto a new `vector`, and return when done. This is the way of the [imperative programmer][8].

Let's continue with some more functionality. We want to count the number of vowels in each word. Check out this test:

```cpp
TEST_METHOD(Vowels)
{
	WordListHelper helper{};
	std::vector < std::string > input{ "aether", "io", "queueing" };
	auto lengths = helper.vowels(input);

	Assert::AreEqual(3, lengths.at(0));
	Assert::AreEqual(2, lengths.at(1));
	Assert::AreEqual(5, lengths.at(2));
}
```

The pattern needs to be a little different here: we're going to have to loop through each string now (i.e. we have nested `for`-statements). Update the class, write the shell method, and make sure your old test passes (and the new one fails!):

```cpp
class WordListHelper
{
public:
	WordListHelper();
	std::vector<int> counts(const std::vector<std::string> words);
	std::vector<int> vowels(const std::vector<std::string> words);
}

std::vector<int> WordListHelper::vowels(const std::vector<std::string> words)
{
	std::vector<int>{ 42 };
}
```

Here's the most straightforward implementation I can think of:

```cpp
std::vector<int> WordListHelper::vowels(const std::vector<std::string> words)
{
	std::vector<int> out{};
	for each (auto word in words)
	{
		int count{ 0 };
		for each (auto chr in word)
		{
			 count += chr == 'a' || chr == 'e' || chr == 'i' || chr == 'o' || chr == 'u';
		}
		out.push_back(count);
	}
	return out;
}
```

OK, so we're starting to get some [code smells][9] here. Two observations come to mind. First, it's not immediately obvious what our function is doing because we've nested two `for`-loops. We should strive to make our code short, crisp, and very simple. Second, we're repeating ourselves here:

```cpp
std::vector<int> out{};
for each (auto word in words)
{
	int some_int{ 0 };
	// ...
	out.push_back(some_int);
}
return out;
```

This pattern is going to keep coming up every time we come up with a new word statistic. "Yeah," you may be thinking, "but its a `for`-loop, how are you going to abstract THAT out?"

As a little motivation for some soul searching, let's add one more method that counts consonants:

```cpp
TEST_METHOD(Consonants)
{
	WordListHelper helper{};
	std::vector < std::string > input{ "Hampsthwaite", "rythms", "strudel" };
	auto lengths = helper.consonants(input);

	Assert::AreEqual(8, lengths.at(0));
	Assert::AreEqual(6, lengths.at(1));
	Assert::AreEqual(5, lengths.at(2));
}
```

You might be tempted to copy and paste the vowels method and insert a bang (`!`):

```cpp
std::vector<int> WordListHelper::consonants(const std::vector<std::string> words)
{
	std::vector<int> out{};
	for each (auto word in words)
	{
		int count{ 0 };
		for each (auto chr in word)
		{
			count += !(chr == 'a' || chr == 'e' || chr == 'i' || chr == 'o' || chr == 'u');
		}
		out.push_back(count);
	}
	return out;
}
```

Or even hack this mess together:

```cpp
std::vector<int> WordListHelper::consonants(const std::vector<std::string> words)
{
	auto lengths = counts(words);
	auto vowelCounts = vowels(words);
	std::vector<int> out{};
	for (int index = 0; index < words.size(); index++) {
		out.push_back(lengths[index] - vowelCounts[index]);
	}
	return out;
}
```

That's enough. Let's spice this class up with some functional programming.

# Maps, functors, and predicates, oh my!
Alright, we've already got our tests and implementations. In the spirit of the [test driven development][10] mantra "red, green, refactor!" Let's do some refactoring.

We noticed that we are writing out the same loop-and-accumulate pattern over and over again. Let's consolidate it all into a [map][11] function:

```cpp
std::vector<int> WordListHelper::map_word(const std::vector<std::string> words,
		std::function<int(std::string)> functor) {
	std::vector<int> out{};
	for each (auto word in words)
	{
		out.push_back(functor(word));
	}
	return out;
}
```

We just passed a function into a function. This allows us to write the logic that loops and accumulates the *map* once, and focus on what our `functor` is doing.

Our `WordListHelper::counts` has a one-liner implementation (thanks to lambdas):

```cpp
std::vector<int> WordListHelper::counts(const std::vector<std::string> words)
{
	return this->map_word(words, [](std::string word) { return word.length(); });
}
```

Our `map_word` takes a `std::function<int(std::string)>` functor. We could have taken the time to write this function out in the usual way:

```cpp
int boring_functor(std::string word)
{
	return word.length();
}
```

But the lambda is extremely compact and expressive. The [syntax][12] is pretty easy once you get the hang of it. Our lambda,

```cpp
[](std::string word) { return word.length(); }
```

contains four important components:

* `[]` is the *capture-list*. Basically, a capture is anything that needs to be dragged along with the lambda when it eventually gets invoked. We'll see an example of this shortly.
* `(std::string word)` is the parameter list, which is just like any other function's parameter list. It contains pairs of types and names.
* `{ return word.length(); }` is the body of the function.
* *Implicit* in this definition is the return type. This can be determined by the return of the function body in most cases.

# Mapping characters
Now let's have a crack at building a map for the characters within each word. This will be very useful when we refactor our `vowels` and `consonants` functions:

```cpp
class WordListHelper
{
public:
	WordListHelper();
	std::vector<int> counts(const std::vector<std::string> words);
	std::vector<int> vowels(const std::vector<std::string> words);
	std::vector<int> consonants(const std::vector<std::string> words);
private:
	std::vector<int> map_word(const std::vector<std::string> words,
			std::function<int(std::string)> functor);
	std::vector<int> map_char(const std::vector<std::string> words,
			std::function<bool(char)> predicate);
};
```

`vowels` and `consonants` functions are slight variations of each other. Consider the following map:

```cpp
std::vector<int> WordListHelper::map_char(const std::vector<std::string> words,
		std::function<bool(char)> predicate) {
	auto char_count = [&](std::string word){
		int count{ 0 };
		for each (auto chr in word)
		{
			count += predicate(chr);
		}
		return count;
	};
	return this->map_word(words, char_count);
}
```

A [predicate][13] is a functor that returns a `bool`. Basically, we pass in an arbitrary predicate, count the number of hits (for a word!) and feed our `map_word` function. We still have no repeated code here--and it's much clearer (after a bit of practice with functional programming) what's going on here.

Notice that the *capture-list* is no longer empty. The special list `[&]` captures all [automatic][14] variables. We need to pull in the `predicate` argument here, and capturing is one easy way to make it work. There are plenty of other possibilities that offer some trade-offs (not within the scope of this introduction).

# The payoff
So what do we get for abstracting away all of this iterating and accumulating? We hard-code a predicate `is_vowel`:

```cpp
WordListHelper::WordListHelper() {
	this->is_vowel = [](char c) { return c == 'a' || c == 'e' || c == 'i' || c == 'o' || c == 'u';  };
}
```

Now check out our new implementations for consonant and vowels.

```cpp
std::vector<int> WordListHelper::vowels(const std::vector<std::string> words)
{
	return this->map_char(words, this->is_vowel);
}

std::vector<int> WordListHelper::consonants_fn(const std::vector<std::string> words)
{
	return this->map_char(words, [&](char c) { return !is_vowel(c); });
}
```

The difference is huge.

*Update:* Thanks to [Code Node][15] (Twitter [@meetingcpp][16]) for pointing out that we can go even further with our refactor. Two `std::` C++11 functions allow us to throw out both `map_char` and `map_word`:

* `std::count_if` will do the dirty work of applying a predicate and counting how many times `true` is returned (this replaces `map_char`)
* `std::transform` is a `map` function (this replaces `map_word`)

An example is available on [github][7]:

```cpp
std::vector<int> WordListHelper::vowels_fn2(const std::vector<std::string> words)
{
	std::vector<int> result{};
	result.resize(words.size());
	std::transform(words.begin(), words.end(), result.begin(),
		[&](std::string word){
			return std::count_if(word.cbegin(), word.cend(), this->is_vowel);
		});
	return result;
}
```

For more information, see the cppreference on [count_if][17] and [transform][18].

# Conclusion

It is vastly easier to extend this class to many other kinds of word statistics by focusing on the functors rather than all the looping. All of the benefits of abstraction that we're used to still apply here: since our loop-and-accumulate map functions are implemented in a single place (i.e. not copied and pasted all over the place with small variations), we could quite easily modify our class for lazy evaluation or parallelize.

The decision on whether to use the lambda syntax or the old-school function declaration/function pointer syntax is largely a style point. Lambdas are certainly some welcome syntactic sugar to C++11. The larger point is that they get us all thinking about functions as first-class citizens, and therefore as prime candidates for abstraction. The results can be stunning.

# Miscellany

* All of the source is available at [https://github.com/JLospinoso/LambdasCpp11][7] as a VS 2013 Solution.
* *Thanks to [John Pavley][6] for his excellent meme.*
