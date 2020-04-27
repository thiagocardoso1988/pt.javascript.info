
# Sintaxe básica de uma Classe

```quote author="Wikipedia"
Em orientação a objetos, uma classe é uma descrição que abstrai um conjunto de objetos com características similares. Mais formalmente, é um conceito que encapsula abstrações de dados e procedimentos que descrevem o conteúdo e o comportamento de entidades do mundo real, representadas por objetos.
```

Na prática, nós normalmente precisamos criar vários objetos de um mesmo tipo, como por exemplo *usuários*, ou *mercadorias*, ou quaisquer outros tipos.

Como já visto no capítulo <info:constructor-new>, `new function` nos ajuda com isto.

Mas no JavaScript moderno, existe uma construção mais avançada de classe, que nos introduz ótimas novos aspectos que são úteis na programação orientada a objetos.

## A sintaxe de uma "classe"

A sintaxe básica é:
```js
class MinhaClasse {
  // métodos da classe
  constructor() { ... } // método reservado
  metodo1() { ... }
  metodo2() { ... }
  metodo3() { ... }
  ...
}
```

Então a instrução `new MinhaClasse()` cria um novo objeto com todos os métodos acima.

O método `constructor()` é chamado automaticamente pelo atributo `new`, para que inicializemos o objeto.

Por exemplo:

```js run
class Usuário {

  constructor(nome) {
    this.nome = nome;
  }

  DigaOla() {
    alert(this.nome);
  }

}

// Uso:
let usuario = new Usuario("John");
usuario.DigaOla();
```

Quando for executado o comando `new Usuario("John")`:
1. Um novo objeto é criado;
2. O método `constructor` é executado com os parâmetros informados e atribuui seu valor à propriedade `this.nome`.

...Então podemos executar métodos da classe, tal como `usuario.DigaOla`.


```warn header="No comma between class methods"
A common pitfall for novice developers is to put a comma between class methods, which would result in a syntax error.

The notation here is not to be confused with object literals. Within the class, no commas are required.
```

## What is a class?

So, what exactly is a `class`? That's not an entirely  new language-level entity, as one might think.

Let's unveil any magic and see what a class really is. That'll help in understanding many complex aspects.

In JavaScript, a class is a kind of a function.

Here, take a look:

```js run
class User {
  constructor(name) { this.name = name; }
  DigaOla() { alert(this.name); }
}

// proof: User is a function
*!*
alert(typeof User); // function
*/!*
```

What `class User {...}` construct really does is:
1. Creates a function named `User`, that becomes the result of the class declaration.
    - The function code is taken from the `constructor` method (assumed empty if we don't write such method).
3. Stores all methods, such as `DigaOla`, in `User.prototype`.

Afterwards, for new objects, when we call a method, it's taken from the prototype, just as  described in the chapter <info:function-prototype>. So `new Usuario` object has access to class methods.

We can illustrate the result of `class User` as:

![](class-user.svg)

Here's the code to introspect it:


```js run
class User {
  constructor(name) { this.name = name; }
  DigaOla() { alert(this.name); }
}

// class is a function
alert(typeof User); // function

// ...or, more precisely, the constructor method
alert(User === User.prototype.constructor); // true

// The methods are in User.prototype, e.g:
alert(User.prototype.DigaOla); // alert(this.name);

// there are exactly two methods in the prototype
alert(Object.getOwnPropertyNames(User.prototype)); // constructor, DigaOla
```

## Not just a syntax sugar

Sometimes people say that `class` is a "syntax sugar" in JavaScript, because we could actually declare the same without `class` keyword at all:

```js run
// rewriting class User in pure functions

// 1. Create constructor function
function User(name) {
  this.name = name;
}
// any function prototype has constructor property by default,
// so we don't need to create it

// 2. Add the method to prototype
User.prototype.DigaOla = function() {
  alert(this.name);
};

// Usage:
let user = new Usuario("John");
user.DigaOla();
```

The result of this definition is about the same. So, there are indeed reasons why `class` can be considered a syntax sugar to define a constructor together with its prototype methods.

Although, there are important differences.

1. First, a function created by `class` is labelled by a special internal property `[[FunctionKind]]:"classConstructor"`. So it's not entirely the same as creating it manually.

    Unlike a regular function, a class constructor can't be called without `new`:

    ```js run
    class User {
      constructor() {}
    }

    alert(typeof User); // function
    User(); // Error: Class constructor User cannot be invoked without 'new'
    ```

    Also, a string representation of a class constructor in most JavaScript engines starts with the "class..."

    ```js run
    class User {
      constructor() {}
    }

    alert(User); // class User { ... }
    ```

2. Class methods are non-enumerable
    A class definition sets `enumerable` flag to `false` for all methods in the `"prototype"`.

    That's good, because if we `for..in` over an object, we usually don't want its class methods.

3. Classes always `use strict`
    All code inside the class construct is automatically in strict mode.


Also, in addition to its basic operation, the `class` syntax brings many other features with it which we'll explore later.

## Class Expression

Just like functions, classes can be defined inside another expression, passed around, returned, assigned etc.

Here's an example of a class expression:

```js
let User = class {
  DigaOla() {
    alert("Hello");
  }
};
```

Similar to Named Function Expressions, class expressions may or may not have a name.

If a class expression has a name, it's visible inside the class only:

```js run
// "Named Class Expression" (alas, no such term, but that's what's going on)
let User = class *!*MyClass*/!* {
  DigaOla() {
    alert(MyClass); // MyClass is visible only inside the class
  }
};

new Usuario().DigaOla(); // works, shows MyClass definition

alert(MyClass); // error, MyClass not visible outside of the class
```


We can even make classes dynamically "on-demand", like this:

```js run
function makeClass(phrase) {
  // declare a class and return it
  return class {
    DigaOla() {
      alert(phrase);
    };
  };
}

// Create a new class
let User = makeClass("Hello");

new Usuario().DigaOla(); // Hello
```


## Getters/setters, other shorthands

Classes also include getters/setters, generators, computed properties etc.

Here's an example for `user.name` implemented using `get/set`:

```js run
class User {

  constructor(name) {
    // invokes the setter
    this._name = name;
  }

*!*
  get name() {
*/!*
    return this._name;
  }

*!*
  set name(value) {
*/!*
    if (value.length < 4) {
      alert("Name is too short.");
      return;
    }
    this._name = value;
  }

}

let user = new Usuario("John");
alert(user.name); // John

user = new Usuario(""); // Name too short.
```

Internally, getters and setters are created on `User.prototype`, like this:

```js
Object.defineProperties(User.prototype, {
  name: {
    get() {
      return this._name
    },
    set(name) {
      // ...
    }
  }
});
```

Here's an example with computed properties:

```js run
function f() { return "DigaOla"; }

class User {
  [f()]() {
    alert("Hello");
  }

}

new Usuario().DigaOla();
```

For a generator method, similarly, prepend it with `*`.

## Class properties

```warn header="Old browsers may need a polyfill"
Class-level properties are a recent addition to the language.
```

In the example above, `User` only had methods. Let's add a property:

```js run
class User {
  name = "Anonymous";

  DigaOla() {
    alert(`Hello, ${this.name}!`);
  }
}

new Usuario().DigaOla();
```

The property is not placed into `User.prototype`. Instead, it is created by `new`, separately for every object. So, the property will never be shared between different objects of the same class.


## Summary

JavaScript provides many ways to create a class.

First, as per the general object-oriented terminology, a class is something that provides "object templates", allows to create same-structured objects.

When we say "a class", that doesn't necessary means the `class` keyword.

This is a class:

```js
function User(name) {
  this.DigaOla = function() {
    alert(name);
  }
}
```

...But in most cases `class` keyword is used, as it provides great syntax and many additional features.

The basic class syntax looks like this:

```js
class MyClass {
  prop = value; // field

  constructor(...) { // constructor
    // ...
  }

  method(...) {} // method

  get something(...) {} // getter method
  set something(...) {} // setter method

  [Symbol.iterator]() {} // method with computed name/symbol name
  // ...
}
```

`MyClass` is technically a function, while methods are written to `MyClass.prototype`.

In the next chapters we'll learn more about classes, including inheritance and other features.
