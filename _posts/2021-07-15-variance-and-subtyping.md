---
title: "Variance and Subtyping"
date: 2021-07-15
tags: [Type Theory, Software Design]
---

Variance is a concept in type systems, especially those with subtyping. Keeping variance in mind when working with advanced type-level machinery in languages is quite helpful, but it applies a lot to simpler concepts too, such as when you are designing classes and interfaces. In this post, we will get an understanding of what variance is and how we can use it to our advantage to write correct code.

# Variance of Type Parameters

## Covariance

Consider the following classes in a statically-typed OOP language; We will be using TypeScript syntax, but don't run it in TypeScript, it won't work.

```ts
class Animal {
    public isLiving(): boolean {
        return true;
    }
}

class Dog extends Animal {
    public woof(): string {
        return 'woof';
    }
}

class Cat extends Animal {
    public meow(): string {
        return 'meow';
    }
}

class ReadBox<T> {
    private value: T;
    public constructor(value: T) {
        this.value = value;
    }
    public getValue(): T {
        return this.value;
    }
}
```

As a refresher, we write `Dog <: Animal` to mean `Dog` is a subtype of (aka "extends" in TypeScript) `Animal`.

Then, the question is, does it follow that `ReadBox<Dog>` is a subtype of `ReadBox<Animal>`? Well, if we want this to be true, it should be that whenever we want to use a `ReadBox<Animal>`, we can instead give it a `ReadBox<Dog>` and everything should work.

```ts
function isLivingInBox(box: ReadBox<Animal>) {
    return box.getValue().isLiving();
}

const dogBox: ReadBox<Dog> = new ReadBox(new Dog());
isLivingInBox(dogBox);
```

And we can see that there really is nothing that stops this from being correct. No matter what, as long as the class extends `Animal` and therefore has the `isLiving` method, it works. Because of the fact that for types `T, U` where `T <: U`, it follows that `ReadBox<T> <: ReadBox<U>`, we say that `ReadBox` is *covariant* in its type parameter[^one_param]: it preserves the subtyping relationship.

## Contravariance

Consider the same `Animal` and `Dog` class, but this different `ListenBox` where we can only tell new values are coming in and be notified of when the box's value changes:

```ts
class ListenBox<T> {
    private listener: (value: T) => void;
    public constructor(listener: (value: T) => void) {
        this.listener = listener;
    }
    public tell(value: T): void {
        this.listener(value);
    }
}
```

Is `ListenBox` covariant in its type parameter?

```ts
const dogBox: ListenBox<Dog> = new ListenBox((newDog) => {
    console.log(newDog.woof());
});

dogBox.tell(new Dog()); // Works fine here...

const animalBox: ListenBox<Animal> = dogBox; // If we assume ListenBox is covariant, this is valid.
animalBox.tell(new Cat()); // Oh no!
```

As you can see, the answer is no. If `ListenBox<Dog> <: ListenBox<Animal>` then we can assign `dogBox` to `animalBox`. But then because `animalBox.tell` accepts `value` of type `Animal`, we can pass in a `Cat`, making the `listener` be called with a `Cat`, which does **not** have the `woof` method. 

So, `ListenBox` is not covariant in its type parameter. But we can salvage something different from this, where it turns out that whenever we want to use a `ListenBox<Dog>`, we can use a `ListenBox<Animal>` and everything works fine:

```ts
const animalBox: ListenBox<Animal> = new ListenBox((newAnimal) => {
    console.log(newAnimal.isLiving());
});

const dogBox: ListenBox<Dog> = animalBox;
dogBox.tell(new Dog()); // Dog has isLiving.

const catBox: ListenBox<Cat> = animalBox;
catBox.tell(new Cat()); // Cat has isLiving.
```

The code above works just fine when instead of saying that `ListenBox` is covariant, we say that it is *contravariant* in its type parameter. This means that for types `T, U` where `T <: U`, we have that `ListenBox<U> <: ListenBox<T>`. The subtyping relationship is reversed, hence the "contra".

## Invariance

Now, for the kind of box that you might see the most often, one where you can read from and write to:

```ts
class Box<T> {
    private value: T;
    public constructor(value: T) {
        this.value = value;
    }
    public getValue(): T {
        return this.value;
    }
    public setValue(value: T): void {
        this.value = value;
    }
}
```

Is this covariant? Well, this example says no:

```ts
const dogBox: Box<Dog> = new Box(new Dog());
const animalBox: Box<Animal> = dogBox; // If we assume Box is covariant, this is valid.
animalBox.setValue(new Cat());
dogBox.getValue().woof(); // Oh no, not again!
```

Since `dogBox` and `animalBox` are aliases of the same `Box`, by assigning `dogBox` to `animalBox`, we are able to put a `Cat` in it via `animalBox`, which once we retrieve it later using `dogBox`, we think it's a `Dog` due to the types, but it is not[^cov_arrays].

So what about contravariance? Well, it turns out all you need to due is rearrange the lines and the types and you run into the exact same problem:

```ts
const animalBox: Box<Animal> = new Box(new Animal());
animalBox.setValue(new Cat());
const dogBox: Box<Dog> = animalBox; // If we assume Box is contravariant, this is valid.
dogBox.getValue().woof(); // Oh no, not a third time!
```

So our `Box` is neither covariant nor contravariant. They are invariant: no matter the relationship between two different types `T, U`, there will never be a subtyping relationship between `Box<T>` and `Box<U>`.

## Bivariance

So we looked at covariance, which preserves the subtyping relationship. We looked at contravariance, which reverses it. Then, we looked at invariance, which completely removes it. You might be wondering whether there is something that allows the type parameter to go both ways.

This is known as *bivariance* and it is very spooky: it only works if there's never any values of that type parameter.

```ts
class PhantomBox<T> {
    // That's it.
}

const animalBox: PhantomBox<Animal> = new PhantomBox(); 
const dogBox: PhantomBox<Dog> = new PhantomBox(); 
const catBox: PhantomBox<Cat> = new PhantomBox();

const animalBox2: PhantomBox<Animal> = dogBox; // Works in preserving the relationship!
const dogBox2: PhantomBox<Dog> = animalBox; // Works in reversing the relationship!
const catBox2: PhantomBox<Cat> = dogBox; // Works in doing absolutely crazy stuff!
```

## Function Types

To facilitate later understanding, we also want to talk about variance in function types.

Let's suppose that we have these function types[^ts_fn], which corresponds to the type of the function take parameters of type `A1` to `An` and returns type `Ret`:

```ts
type Function0<Ret>
type Function1<A1, Ret>
type Function2<A1, A2, Ret>
type Function3<A1, A2, A3, Ret>
// And so on...
```

What is the variance on these type parameters? Let's consider that we are writing the function with type `Function1<Dog, Animal>` for perhaps a callback. This means that someone else will pass us the `Dog` that we can use. But, we don't have to use all of the `Dog`, for example, we only use `isLiving`. This means that `Function1<Animal, Animal>` will also work in place of the previous type for the callback. Therefore, `Function1` is contravariant in the type of its first type parameter. Now if we consider from the perspective of the person calling our callback, they are expecting to receive an `Animal`. This means that it is perfectly fine to use a subtype of `Animal`, such as `Dog`. Therefore, `Function1` is covariant in the type of its return type parameter.

In general, for functions, the types of the parameters are contravariant, while the type of the return is covariant. This leads to a guideline for the variance of type parameters in general: if a type has a value that is to be "produced", it is covariant. If it is to be "consumed", it is contravariant. If both, it is invariant; if neither, it is bivariant[^position]. You can see with our boxes that `ReadBox` only produced values via `getValue`, `ListenBox` only consumed values via `tell` and `listener`, and `Box` produced and consumed values with `getValue` and `setValue`.

## Variance Annotations

In practice, the variance of type parameters in a class or interface that you declare is very language-dependent. Some languages always have invariant type parameters. Others allow you to specify the variance of type parameters.

Here's a little summary:

- **Java**: Invariant type parameters, but allows specifying variance at use-sites. This means you can actually change the variance of a type parameter depending on how you want to use it at a certain place and time. You can look up Java's wildcards for more information.

- **Kotlin**, **Scala**, **C#**, **OCaml**: Allows specifying invariant, covariant, or contravariant type parameters. These languages will also check that these type parameters are used in such a way that they do not contradict their variance annotation.

- **TypeScript**: An unsound mess. Variance is not checked at all, but instead properties are checked if they are assignable to each other. This sometimes ensures variance is respected, sometimes it doesn't.

# Variance in Software Design

## Overriding Methods

Inheritance is what allows us to make classes (or interfaces) that extend other classes (or interfaces). When a class `B` inherits from a class `A`, languages will now consider `B` to be a subtype of `A`. Of course, once we have subtyping we have to get to variance. Specifically, how much can we override the type signatures of the methods of a class?

If we look at the bigger picture, we realize that a class declaration isn't too different from assigning values to variables. The parent class tells you that these methods have certain types, and we can "re-assign" or override these methods in a subclass.

```ts
class AnimalBox {
    // It's like declaring get: Function0<Animal> = ...
    public get(): Animal {
        // ...
    }
    public set(value: Animal): void {
        // ...
    }
}

class DogBox {
    // It's like re-assigning get = ...
    public get(): Dog {
        // ...
    }
    public set(value: Object): void {
        // ...
    }
}
```

And we can see indeed that our `DogBox` is using the concept of variance. In the case of `get`, it is almost as if we are assigning a `Function0<Dog>` to a `Function0<Animal>`, which is valid because functions are covariant in the return type. Similarly, for `set`, it is like we are assigning `Function1<Object, void>` to `Function1<Animal, void>`, which is valid because `Animal <: Object` and functions are contravariant in the parameter types.

## Liskov's Substitution Principle

Liskov's Substitution Principle, the L in SOLID: the principle says that if a property about a type `T` is true, then it must also be true for a subtype of `T`[^liskov]. It is from this principle that covariance and contravariance of methods were adopted in newer OOP languages.

But the principle does not just apply to the types of objects that you type into the source code and is checked by the compilers. We have to consider things that are more than just the language, we have to consider things that you might instead document in a comment: refinements, preconditions, postconditions, and invariants.

### Refinements

When your language does not provide a way (or an easy way) to *refine* your types[^refinements], which is to make them more specific for the domain you are working on, you might instead document those refinements. For example, consider a method that returns a `String` value that must be a valid email address. This means that, by return types being covariant, methods which override that method must only return `String` values that are a valid email address or stronger (perhaps, a valid Gmail address).

In the opposite direction, perhaps your method takes a tuple `(Int, Int)` which must have `x^2 + y^2 = 1`. A subclass can override that method and allow a weaker constraint if it wanted to, such as `x^2 + y^2 <= 1`. It cannot, however, allow a stronger constraint, because this would break the contravariance of method parameter types.

### Preconditions and Postconditions

Preconditions are requirements that a method wants to be true about the state of the program before it runs. For example, you might ask that before calling `advance` on a parser that you are not at the end of the string. Preconditions are contravariant[^sets_conds]. They can only be weakened when overridden, not strengthened. You might for example have a parser implementation that allows for streaming parsing, in which case calls to `advance` may be called past the string in buffer currently.

Likewise, postconditions are things about the state of the program that must be true after the method runs. They are covariant, so they can only be strengthened when overridden, not weakened.

### Invariants

Invariants are always true statements about your program that must never be broken. You can imagine that this is, well, invariant. For example, a parser invariant could be that we never go backwards in the input of the parser.

A special type of invariant that always apply is the idea of encapsulation. It says that if a class allows its state to be modified through a set of methods, then those are the only methods that can modify that state. This means that a subclass that introduces new behavior cannot modify itself in ways that are not allowed by the superclass i.e. it must use the methods provided by the superclass to do so. This rule is called the "history rule". Of course, the subclass itself can define new mutable state and corresponding methods separately from the superclass. 

# Conclusion

Hopefully this post gives you a good understanding of what variance is in terms of OOP languages and how it applies to OOP design.

For those interested in theory, the concept of variance comes from category theory, where one would study covariant and contravariant functors. Those who know a bit of Haskell would also encounter them there! 

---

[^one_param]: We can say "its type parameter" because it only has one. You can imagine some type where only some parameters are covariant.

[^cov_arrays]: It turns out that quite a few languages (Java, C#, TypeScript, etc.) have covariant arrays. You can think of our `Box` as an array with just one item at all times. As you can see, covariant arrays are a very bad idea! They should be invariant.

[^ts_fn]: In TypeScript, the actual syntax for function types is `(arg: A1) => Ret`, but this syntax is not in line with the syntax with angle brackets, so we won't use it in this post.

[^position]: This is just a guideline. It starts to become unwieldy when you have functions that are higher-order than just callbacks. For a more rigid approach, consider function parameters to be "negative" and function returns to be "positive", and that each time you introduce a type in a negative position, you invert all of the types inside. Then, types that are in only positive positions are covariant, in only negative positions are contravariant, in both are invariant, and in neither are bivariant. As an example, the type (in Haskell-like syntax) `a -> (b -> a) -> ((c -> a) -> b) -> c` becomes `-a -> (+b -> -a) -> ((-a -> +c) -> -b) -> +c` which means `a` is contravariant, `b` is invariant, and `c` is covariant.

[^liskov]: This is indeed a pretty vague definition. The [Wikipedia page](https://en.wikipedia.org/wiki/Liskov_substitution_principle) goes much more in depth about the formalities.

[^refinements]: Actually, these refinements we are referring to are usually thought of as preconditions on the parameters and postconditions on the returns. But I feel it makes more sense when looked as if they were actual types that a language just happens to not be able to express. Look up refinement types!

[^sets_conds]: You can imagine that every function type actually takes another type parameter, one for a set of preconditions, whose subtyping rules follow that of the superset relation. Likewise, postconditions would follow the subset relation.
