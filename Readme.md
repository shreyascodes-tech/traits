# Traits for TypeScript
it is exactly what it sounds like `Traits` for `TypeScript` :D

## Context
[@techsavvytravvy](https://twitter.com/techsavvytravvy)
started [this discussion](https://twitter.com/techsavvytravvy/status/1707247698120728889) on Twitter about traits in TypeScript. I thought it would be fun to try to implement traits in TypeScript.

Traits are a way to implement multiple inheritance without the problems of multiple inheritance. Traits are a way to share code between classes. Traits are similar to interfaces, but they can also contain code.
I am trying to implement traits from the [Rust programming language](https://www.rust-lang.org/) in TypeScript.

## Design

Traits can be implemented using the `@trait()` decorator. Classes and interfaces can implement traits using the `@impl()` decorator. The `@impl()` decorator takes a list of trait types as a parameter.

To use a trait in a class, you can use the `TraitImpl.use()` function. The `TraitImpl.use()` function takes a callback function as a parameter. The callback function is executed with the implementation of the trait bound to the `this` keyword.

## **Example**

The following example shows how to implement a `Shape` trait and a `Printable` trait:

```typescript
@trait()
export class Shape {
  area!: () => number;
}

@trait()
export class Printable {
  print!: () => void;
}
```

The following example shows how to implement `Printable` for the `Shape` trait:

```typescript
@impl(Printable).for(Shape)
export class ShapePrintableImpl {
  print() {
    console.log(`Area: ${this.area()}`);
  }
}
```

The following example shows how to implement a `CircleShapeImpl` class that implements the `Shape` trait:

```typescript
@impl(Shape).for(Circle)
export class CircleShapeImpl {
  radius!: number;

  area() {
    return Math.PI * this.radius ** 2;
  }
}
```

The following example shows how to use the `CircleShapeImpl` class to print a circle to the console:

```typescript
CircleShapeImpl.use(() => {
  ShapePrintableImpl.use(() => {
    const circle = new Circle();
    circle.radius = 10;
    circle.print();
  });
});
```

we could maybe generate this code as well ¯\_(ツ)_/¯

## **Code gen**

The generated code could look like this:

```typescript
declare module "./struct.ts" {
  export interface Circle extends Shape {}
  export interface CircleShapeImpl extends Circle {}
  export interface ShapePrintableImpl extends Shape, Printable {}
  export namespace ShapePrintableImpl {
    export function use(f: () => void): void;
  }
  export interface Shape extends Printable {}
}
```

And the generated use functions could look like this:

```typescript
CircleShapeImpl.use = function (f) {
  Circle.prototype.area = CircleShapeImpl.prototype.area;
  f();
  delete Circle.prototype.area;
};

ShapePrintableImpl.use = function (f) {
  Circle.prototype.print = ShapePrintableImpl.prototype.print;
  f();
  delete Circle.prototype.print;
};
```

The final output could look like this:

```typescript
export class Shape {
  area!: () => number;
}

export class Circle {
  radius!: number;
}
export class CircleShapeImpl {
  area() {
    return Math.PI * this.radius ** 2;
  }
}

export class Printable {
  print!: () => void;
}
export class ShapePrintableImpl {
  print() {
    console.log(this.area());
  }
}

// Generated code, this will go in a different file
export interface Circle extends Shape {}
export interface CircleShapeImpl extends Circle {}
export interface ShapePrintableImpl extends Shape, Printable {}
export namespace ShapePrintableImpl {
  export function use(f: () => void): void;
}
export namespace CircleShapeImpl {
  export function use(f: () => void): void;
}

export interface Shape extends Printable {}

CircleShapeImpl.use = function (f) {
  Circle.prototype.area = CircleShapeImpl.prototype.area;
  f();
  // @ts-ignore ["we bend it to our will"](https://twitter.com/techsavvytravvy/status/1707418798838423679)
  delete Circle.prototype.area;
};

ShapePrintableImpl.use = function (f) {
  Circle.prototype.print = ShapePrintableImpl.prototype.print;
  f();
  // @ts-ignore ["we bend it to our will"](https://twitter.com/techsavvytravvy/status/1707418798838423679)
  delete Circle.prototype.print;
};

// Generated code end

CircleShapeImpl.use(() => {
  ShapePrintableImpl.use(() => {
    const circle = new Circle();
    circle.radius = 10;
    circle.print();
  });
});
```
> This is valid typescript code and you can run it with type safety and all the intellisense you would expect from TypeScript.

I dont know if this is the best way to implement traits in TypeScript. I am open for suggestions.