---
layout: post
title:  "Tests readable like well-written prose.<br/>Part 1 - AAA / GWT rule."
date:   2013-12-17 23:30:00
categories: tests, cleancode
---

Spiltting a test into three code blocks, where you:
- **arrange** test data and [SUT](http://en.wikipedia.org/wiki/System_under_test),
- **act** on SUT
- **assert** output correctness

improves its readablity.

Just look at this simple test before introducing clear test block boundaries:

{% highlight java %}
@Test
public void shouldAddTwoNumbers() {
	int firstNumber = 1;
	int secondNumber = 2;
	Calculator calculator = new Calculator();
	int result = calculator.add(firstNumber, secondNumber);
	assertThat(result).isEqualTo(3);
}
{% endhighlight %}

and compare it to one with blocks separated using code comments:


{% highlight java %}
@Test
public void shouldAddTwoNumbers() {
	// arrange
	int firstNumber = 1;
	int secondNumber = 2;
	Calculator calculator = new Calculator();
	// act
	int result = calculator.add(firstNumber, secondNumber);
	// assert
	assertThat(result).isEqualTo(3);
}
{% endhighlight %}

Although the second test is a bit longer, it is more readable.

It is easier to say:
- what is tested
- what data is used as SUT input
- what is checked as part of particular test case (what assertions are applied to SUT output)

What is very important - you should use each block only once per test case. By following this rule you will end up with all tests having exact same structure making them easier to follow.

Do you remember what [Ward Cunningham](http://c2.com/ward/) (inventor of the Wiki, inventor of Fit, co-inventor of eXtreme Programming, and much more) said about clean code?

> You know you are working on clean code when each routine you read turns out to be pretty much what you expected.

AAA rule is more popular as 'Given / When / Then' (GWT). Reasoning is exactly the same, but [BDD-like](http://en.wikipedia.org/wiki/Behavior-driven_development) comments are more often used to separate test's code blocks:

{% highlight java %}
@Test
public void shouldAddTwoNumbers() {
	// given
	...
	// when
	...
	// then
	...
}
{% endhighlight %}
