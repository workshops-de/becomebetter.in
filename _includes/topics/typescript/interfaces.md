### Interfaces in TypeScript

Interfaces in TypeScript are a good way to define the shape for an object or class definitions.

It is possible to describe properties and their types inline by using the following syntax.

```ts
const book: {
  isbn: string;
  title: string;
}

book.isbn = '1234-1234-1234-1234'; // ✅
book.title = 123; // 🚫 Error: 123 is not a string
book.unknown = null; // 🚫 Error: this property is not part of book constant
```

This might be handy but is not reusable since each further variable which contains a book has to be typed inline, too.

A better approach is to use TypeScript interfaces which can be reused.

```ts
interface Book {
  isbn: string;
  title: string;
}

const book: Book = {
  isbn: '1234-1234-1234-1234',
  title: 'TypeScript is great'
};

const anotherBook: Book = {
  isbn: '5678-5678-5678-5678',
  title: 'Another great book title'
};
```

It is a common way to name intefaces [UpperCamelCase](https://en.wikipedia.org/wiki/Camel_case) such as `Book` or `BookShelf`.

In TypeScript, only properties an be used which are declared in the interface. It is not allowed to set or read unknown properties, even this would be possible in JavaScript. However properties can be marked as optional to deal with that situation. Optional properties must be named with a `?` suffix such as `pages?`.

```ts
interface Book {
  isbn: string;
  title: string;
  pages?: number;
}
```

Please note that `pages` has no default value. The property is `undefined` if no value is set, otherwise it must be type `number`. To use this property it must be checked if it has been set.

```ts
if (book.pages) {
  // If pages is thuthy than it must be type number
  console.log('Number of pages', book.pages);
}
```

#### Interfaces in Classes

TypeScript classes can implement interfaces, too, which describes the shape of the class and all required or optional properties and even methods. It is possible to implement a various number of interfaces for one class. 

```ts
interface BookShelf {
  getStoredBooks(): Book[]:
}

class Shelf implements BookShelf {

  private books: Book[] = [];

  getStoredBooks(): Book[] {
    return this.books;
  }
}
```

TypeScript will throw a compile error if the method `getStoredBooks()` isn't either implemented or doesn't return an array of books.