---
layout: post
title:  "Tests readable like well-written prose. Part 2 - Test Data Builder pattern"
date:   2014-01-25 18:15:00
categories: blog
tags: tests cleancode design-patterns
---

Lets look at sample Employee class:

	
{% highlight java %}
public class Employee {

	private String firstName;
	private String lastName;
	private Address address;
	private Grade grade;
	private BigDecimal remuneration;

	...
}
{% endhighlight %}

I model classes in a way they are logically valid objects just after creation, so in case of class shown above I would like to:

- treat all fields as not nullable,
- initialise all not nullable fields in a constructor
- make the class immutable

We could then make all fields final, add non-null checks and add a constructor, but constructor with 5 attributes, probably even more in the future, ... hmm.. no.

Lets write (or rather generate) a builder!


{% highlight java %}
public final class Employee {

	private final String firstName;
	private final String lastName;
	private final Address address;
	private final Grade grade;
	private final BigDecimal remuneration;

	private Employee(String firstName, String lastName, Address address, Grade grade, BigDecimal remuneration) {
		checkNotNull(firstName);
		checkNotNull(lastName);
		checkNotNull(address);
		checkNotNull(grade);
		checkNotNull(remuneration);

		this.firstName = firstName;
		this.lastName = lastName;
		this.address = address;
		this.grade = grade;
		this.remuneration = remuneration;
	}

	public String getFirstName() {
		return firstName;
	}

	public String getLastName() {
		return lastName;
	}

	public Address getAddress() {
		return address;
	}

	public Grade getGrade() {
		return grade;
	}

	public BigDecimal getRemuneration() {
		return remuneration;
	}

	public static final class Builder {
		private String firstName;
		private String lastName;
		private Address address;
		private Grade grade;
		private BigDecimal remuneration;

		public Builder withFirstName(String firstName) {
			this.firstName = firstName;
			return this;
		}

		public Builder withLastName(String lastName) {
			this.lastName = lastName;
			return this;
		}

		public Builder withAddress(Address address) {
			this.address = address;
			return this;
		}

		public Builder withGrade(Grade grade) {
			this.grade = grade;
			return this;
		}

		public Builder withRemuneration(BigDecimal remuneration) {
			this.remuneration = remuneration;
			return this;
		}

		public Employee build() {
			return new Employee(firstName, lastName, address, grade, remuneration);
		}
	}
}
{% endhighlight %}

This looks better.

Lets now add some logic - we would like to calculate employee's annual bonus, which value is based on his/her grade.

{% highlight java %}
@DataProvider(name = "remuneration-grade-bonus")
public Object[][] getTestData() {
	return new Object[][]{
			{100000.0, JUNIOR, 0.0},
			{100000.0, REGULAR, 5000.0},
			{100000.0, SENIOR, 10000.0},
			{100000.0, PRINCIPAL, 15000.0}
	};
}

@Test(dataProvider = "remuneration-grade-bonus")
public void shouldCalculateAnnualBonusBasedOnEmployeeGrade(double remuneration, Grade grade, double expectedBonus) {
	// given
	Employee sut = new Employee.Builder()
			.withFirstName("Jan")
			.withLastName("Kowalski")
			.withAddress(new Address.Builder()
					.withCountry("Poland")
					.withCity("Warsaw")
					.withStreet("SomeStreet 12")
					.withZipCode("00-100")
					.build())
			.withGrade(grade)
			.withRemuneration(BigDecimal.valueOf(remuneration))
			.build();

	// when
	BigDecimal annualBonus = sut.calculateAnnualBonus();

	// then
	assertThat(annualBonus.doubleValue()).isEqualTo(expectedBonus, offset(0.01));
}
{% endhighlight %}


We add implementation:

{% highlight java %}
public BigDecimal calculateAnnualBonus() {
	return remuneration.multiply(BigDecimal.valueOf(grade.getBonusPercentage()))
			.divide(BigDecimal.valueOf(100), MathContext.DECIMAL128);
}
{% endhighlight %}

and it works.

{% highlight java %}
Pass	shouldCalculateAnnualBonusBasedOnEmployeeGrade (100000.0, JUNIOR, 0.0)
Pass	shouldCalculateAnnualBonusBasedOnEmployeeGrade (100000.0, REGULAR, 5000.0)
Pass	shouldCalculateAnnualBonusBasedOnEmployeeGrade (100000.0, SENIOR, 10000.0)
Pass	shouldCalculateAnnualBonusBasedOnEmployeeGrade (100000.0, PRINCIPAL, 15000.0)
{% endhighlight %}

Ok, it works, but... test is a bit too long and most of the code is not related to annual bonus calculation in any way.

We had to provide all employee's details (most of which are not connected to tested logic), since otherwise we couldn't create valid Employee object.

How could we do it better then?

- extract employee creation to private method with grade and remuneration arguments
- or like above, but extract it to different class, so it could be reused in different tests (<a target="_blank" href="http://martinfowler.com/bliki/ObjectMother.html">Object Mother</a> pattern)

Yeah..., but these have some limitations too:
- private method in the test cannot be reused in other tests, hence will lead to code duplication and harder maintenance of test code
- Object Mother will be reusable, but this reusability comes at a price - code coupling. Because now many of your tests can use the same method to build test object, any change to this method implementation could affect all tests you wrote before. Object Mother pattern also leads to method explosion - you could very fast end up having:
    - aSeniorEmployee(),
    - aSeniorEmployeeWithRemunerationOf(double),
    - aJuniorEmployeeWithAddress(Address),
    - etc ...
    - or even worse: anEmployee(String, String, String)

I would like the test to look like below - having only given test-scenario related data:

{% highlight java %}
@Test(dataProvider = "remuneration-grade-bonus")
public void shouldCalculateAnnualBonusBasedOnEmployeeGrade(double remuneration, Grade grade, double expectedBonus) {
	// given
	Employee sut = anEmployee().withRemuneration(remuneration).withGrade(grade).build();

	// when
	BigDecimal annualBonus = sut.calculateAnnualBonus();

	// then
	assertThat(annualBonus.doubleValue()).isEqualTo(expectedBonus, offset(0.01));
}
{% endhighlight %}

"Hey, but this is just a builder?!" you could say.

Yes it is (that's why it is named Test Data Builder), but it also has some reasonable default values set, so we don't need to provide values we don't care of. Such builder shouldn't be available in production code off-course, but in test code it makes a lot of sense.

Here is how it could be written:

{% highlight java %}
public final class EmployeeTestDataBuilder {

	private final Address.Builder addressBuilder = new Address.Builder()
			.withCountry("Poland")
			.withCity("Warsaw")
			.withStreet("SomeStreet 12")
			.withZipCode("00-100");

	private final Employee.Builder employeeBuilder = new Employee.Builder()
			.withFirstName("Jan")
			.withLastName("Kowalski")
			.withGrade(Grade.REGULAR)
			.withRemuneration(BigDecimal.valueOf(12000));

	private EmployeeTestDataBuilder() {
	}

	public static EmployeeTestDataBuilder anEmployee() {
		return new EmployeeTestDataBuilder();
	}

	public EmployeeTestDataBuilder withGrade(Grade grade) {
		employeeBuilder.withGrade(grade);
		return this;
	}

	public EmployeeTestDataBuilder withRemuneration(double remuneration) {
		employeeBuilder.withRemuneration(BigDecimal.valueOf(remuneration));
		return this;
	}

	public Employee build() {
		return employeeBuilder
				.withAddress(addressBuilder.build())
				.build();
	}
}
{% endhighlight %}

This is a Test Data Builder.

It has many advantages:
- tests are clean and readable,
- there is no risk of changing default values or adding new data, because all important values used in test are explicitly set there,
- test code is easy to maintain - adding new field to Employee class doesn't force you to modify all existing tests what would be needed in case of constructor based creation or even production-code builder,
- test builder can take values in different form than in your production-code builder or constructor. It is specially useful for dates - you can write for example .withStartDate(2013, 10, 28) and perform conversion in a builder, what will make tests easier to read

The only disadvantage I see is builder code that must be written, but most of IDEs has some plugins to generate them. I use <a target="_blank" href="http://plugins.jetbrains.com/plugin/6585?pr=idea">Builder Generator</a> for Intellij Idea.
