# Builder design pattern

The builder design pattern as its name indicates, helps us to build or create an object.

The Builder design pattern can be likened to an assembly line in a manufacturing process, such as in a car factory. In this setting, a vehicle is assembled incrementally. It starts as a basic frame and, as it progresses down the line, different components like the engine, doors, and tires are added. Each car might receive different features, such as air conditioning or sunroofs, depending on its model.

In a similar vein, the Builder pattern in programming constructs an object in stages. Initially, you have a bare-bones object. Through the builder, you then add attributes one by one, like a title, author, or page count for a book. Just as cars on an assembly line can have various features based on their model, the Builder pattern allows for creating objects with diverse combinations of attributes without needing a separate constructor for each variation. This approach provides a clear and flexible method for constructing complex objects, mirroring the efficiency and customization of an assembly line.

Imagine that you have an object that requires several attributes, and all of them need to be provided by the user, it is very common that the user doesn't know all the attributes at the moment when is filling the object.

Consider a bookstore management system where books are cataloged before their physical arrival. Each book is characterized by attributes such as title, author, number of pages, and price. This scenario often results in situations where we don't physically possess the book yet, but we do have some information about it.

```java
@Data
public class Book {

    private String title;

    private String author;

    private int pages;

    private Money price;
}
```

> @Data is a lombook annotation.

Just for a matter of ease the example we can consider this book as a complex object (in the real world we could have many more attributes for our complex objects).

Because we don't have the real book in our hands we may not know some of the data, also, because books come from different editors, we do receive not the same data from each editor, that is one editor provides the author name, but others don't.

In this scenario if we want to be able to add a book we may need a huge number of constructors for all the data combinations we could receive:

* Constructors with a single attribute.
* Constructors with two attributes.
* Constructors with three attributes.
* The constructor with all four attributes.

Here are the possible combinations:

Constructors with one attribute:

1. Book(String title)
2. Book(String author)
3. Book(int pages)
4. Book(Money price)

Constructors with two attributes:

5. Book(String title, String author)
6. Book(String title, int pages)
7. Book(String title, Money price)
8. Book(String author, int pages)
9. Book(String author, Money price)
10. Book(int pages, Money price)

Constructors with three attributes:

11. Book(String title, String author, int pages)
12. Book(String title, String author, Money price)
13. Book(String title, int pages, Money price)
14. Book(String author, int pages, Money price)

Constructor with all four attributes:

15. Book(String title, String author, int pages, Money price)

For each of these constructors, you'll need to initialize the provided attributes in the constructor and leave the others as null or a default value, as appropriate. Here's an example of what some of these constructors might look like:

```java
public class Book {

    private String title;
    private String author;
    private int pages;
    private Money price;

    public Book(String title) {
        this.title = title;
    }

    public Book(String title, String author) {
        this.title = title;
        this.author = author;
    }

    public Book(String title, String author, int pages) {
        this.title = title;
        this.author = author;
        this.pages = pages;
    }

    // ... other constructors

    public Book(String title, String author, int pages, Money price) {
        this.title = title;
        this.author = author;
        this.pages = pages;
        this.price = price;
    }
}

```

As you can see this is a crazy situation and a lot of useless effort is required.
Don't forget that our example has only 4 attributes, imagine how many constructors may be needed for an object with 15 attributes or more.
Fortunately for us, the `builder design pattern` will save us from this craziness.

Here is the code with `builder pattern`

```java
import lombok.Data;
import org.javamoney.moneta.Money;

@Data
public class Book {

    private String title;

    private String author;

    private int pages;

    private Money price;

    protected Book(String title, String author, int pages, Money price) {
        this.title = title;
        this.author = author;
        this.pages = pages;
        this.price = price;
    }

    public static class Builder {
        private String title;

        private String author;

        private int pages;

        private Money price;

        public Builder() {}

        public Builder withTitle(String title) {
            this.title = title;
            return this;
        }

        public Builder withAuthor(String author) {
            this.author = author;
            return this;
        }

        public Builder withPages(int pages) {
            this.pages = pages;
            return this;
        }

        public Builder withPrice(Money price) {
            this.price = price;
            return this;
        }

        public Book build() {
            return new Book(this.title, this.author, this.pages, this.price);
        }

    }
}

```

I know, let's add notes and explanations on this code.
first of all, we added the protected constructor to restrict access, ensuring that other classes cannot directly instantiate the object.

```java
    // External objects won't be able to construct a book.
    protected Book(String title, String author, int pages, Money price) {
        this.title = title;
        this.author = author;
        this.pages = pages;
        this.price = price;
    }
```

Then we build an internal class named `Builder`:

```java
public static class Builder {
        private String title;

        private String author;

        private int pages;

        private Money price;

        public Builder() {}
}
```

As you can notice this `Builder class` mirrors the `Book class` by containing the same attributes.

Now here comes the important part, let's code a method that sets one of the attributes and returns the `Builder` instance itself.

```java
//we receive the book title and `save` it for latter use.
public Builder withTitle(String title) {
            this.title = title;
            //we don´t have a book, but have a configured builder.
            return this; //returning the builder will allow us to join other attributes to the same builder.
        }
```

Doing this we don't have a book yet, but it sets the stage for us to configure an assembly line capable of adding titles to our books in the future.

let's do the same with all our attributes.

```java
        public Builder withTitle(String title) {
            this.title = title;
            return this;
        }

        public Builder withAuthor(String author) {
            this.author = author;
            return this;
        }

        public Builder withPages(int pages) {
            this.pages = pages;
            return this;
        }

        public Builder withPrice(Money price) {
            this.price = price;
            return this;
        }
```

The `return this` statement in the builder is crucial as it enables chaining, allowing us to seamlessly connect different parts of the builder process.
And finally, we need a way to build the book, because until now we have builders, but not books.

```java
        public Book build() {
            return new Book(this.title, this.author, this.pages, this.price);
        }
```

Ok, sounds good (really?), but how do we use it?
Obviously, building a unit test.

```java
class BookTest {

    @Test
    public void bookTestBuilder() {
        Book book = new Book.Builder()
                .withTitle("Pedro Páramo")
                .withAuthor("Juan Rulfo")
                .withPages(128)
                .withPrice(Money.of(267.29f, "USD"))
                .build();

        System.out.println(book.getPrice().toString());

        assertEquals("Pedro Páramo", book.getTitle());
        assertEquals("Juan Rulfo", book.getAuthor());
        assertEquals(128, book.getPages());
        assertEquals(Money.of(267.29f, "USD"), book.getPrice());
    }
}
```

1. Create the builder
    ```java
    Book.Builder()
    ```
2. Add the attributes we know.
    ```java
    .withTitle("Pedro Páramo")
    .withAuthor("Juan Rulfo")
    .withPages(128)
    .withPrice(Money.of(267.29f, "USD"))
    ```
3. Build the book
    ```java
        .build();
    ```
All in a single step
```java
    Book book = new Book.Builder()
                .withTitle("Pedro Páramo")
                .withAuthor("Juan Rulfo")
                .withPages(128)
                .withPrice(Money.of(267.29f, "USD"))
                .build();
```

Now we have a book that contains only the attributes we know at the moment of its creation.

```java
    Book book = new Book.Builder()
                .withTitle("Pedro Páramo")
                .withAuthor("Juan Rulfo")
                .build();
```

[However, this is just the beginning.](./builder2.md)

