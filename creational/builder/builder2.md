# Builder design pattern

[In the previous article](/creational/builder.md), we discovered the main idea behind the builder pattern and illustrated it with a basic example (probably the most simple way to do it).

But is it a good idea to have our `builder class` inside the `class to build`? I mean is it a good idea to define `Builder` inside `Book.java`?

For teaching purposes and to simplify the explanation, this approach works well. However, in practice, I advocate for separating responsibilities across different classes to promote cleaner code organization.

Therefore, let's proceed by creating the `Builder class` in its own separate Java file.

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Person {

    private String firstName;

    private String lastName;

    private LocalDate dob;
}
```
>Remember we are using lombok annotations.

`Person` is now our complex object, to maintain it clean and simple we need to create the builder class.

```java
public class PersonBuilderAttributes {

    private String firstName;
    private String lastName;
    private LocalDate dob;

    public PersonBuilderAttributes() {}

    public PersonBuilderAttributes withFirstName(String firstName) {
        this.firstName = firstName;
        return this;
    }

    public PersonBuilderAttributes withLastName(String lastName) {
        this.lastName = lastName;
        return this;
    }

    public PersonBuilderAttributes withDob(LocalDate dob) {
        this.dob = dob;

        return this;
    }

    public Person build() {
        return new Person(firstName, lastName, dob);
    }
}
```

Now we have a clean, well organized code, with separated responsibilities (sounds like we are following the *Single responsibility principle*).

Ok, but how do we use it?
Obviously, building a unit test.

```java
class PersonBuilderAttributesTest {

    @Test
    public void build() {
        PersonBuilderAttributes builder = new PersonBuilderAttributes();

        Person person = builder.withFirstName("Emiliano Alberto")
                .withLastName("Méndez")
                .withDob(LocalDate.of(2011, Month.JANUARY, 4))
                .build();

        System.out.println(person);

        assertEquals("Emiliano Alberto", person.getFirstName());
        assertEquals("Méndez", person.getLastName());
        assertEquals(LocalDate.of(2011, Month.JANUARY, 4), person.getDob());
    }
}
```

Great we have learned the `builder design pattern`, but what if we need to add or remove some attributes to the `Person class`?

Simple, you just need to add them to both classes `Person` and `PersonBuilderAttributes` (what a horrible name).
Be careful, that is just the opening door to the legacy hell.

If we need to apply the same changes in different places of our code, some bad smell should come to us. You know I may forget to update the `Builder class` and get and the new attribute becomes useless.

Yes, I know this is overreacting to something that is not really that risky, but I'm a lazy person, so is there a way to avoid copying one line of code?

Yes, let's code it.

```java
public class PersonBuilder {
    private final Person person;

    public PersonBuilder() {
        this.person = new Person();
    }

    public PersonBuilder withLastName(String lastName) {
        person.setLastName(lastName);
        return this;
    }

    public PersonBuilder withFirstName(String firstName) {
        person.setFirstName(firstName);
        return this;
    }

    public PersonBuilder withDob(LocalDate dob) {
        person.setDob(dob);
        return this;
    }

    public Person build() {
        return person;
    }

}
```

A simple change, in our `builder class` we change the attributes list for one instance of our object to build.

~~private String firstName;~~

~~private String lastName;~~

~~private LocalDate dob;~~

private final Person person;

Let's build it's unit test.

```java
class PersonBuilderInternoTest {

    @Test
    void build() {

        Person person = new PersonBuilder()
                .withFirstName("Leonardo")
                .withLastName("Méndez")
                .withDob(LocalDate.of(2015, Month.DECEMBER, 4))
                .build();

        System.out.println("The person is: " + person);

        assertEquals("Leonardo", person.getFirstName());
        assertEquals("Méndez", person.getLastName());
        assertEquals(LocalDate.of(2015, Month.DECEMBER, 4), person.getDob());
    }
}
```

This is happiness, now I just need to add the new attribute in one place.

**Adios...**

Hold on a second. With these new approaches, it seems we're required to have public constructors for our `Person class`. But is it better to have a protected constructor with an inner builder?

It really depends on our specific requirements. If the business logic dictates that it's essential to compel users to create objects solely via the Builder, then yes, an inner builder with a protected or package-private constructor is the only viable option.

Thus, employing the Builder pattern is an elegant, intelligent, and proven solution that enhances the professionalism of our code.

Thank you for your...

Wait, what about scenarios involving inheritance in the object we're building?...




