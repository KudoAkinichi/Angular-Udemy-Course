# JavaScript Refresher: Classes, Properties & More

Angular makes heavy use of **classes** – a feature that's supported by vanilla JavaScript and TypeScript (though TypeScript extends it and adds some extra features, as you'll see).

---

## What is a Class?

A class is essentially a **blueprint for objects**.  
Any **properties** and **methods** defined in the class will exist on all objects that are created based on the class.

### Example (Vanilla JavaScript)

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  greet() {
    console.log('Hi, I am ' + this.name);
  }
}
````

### Instantiating a Class

```javascript
const person1 = new Person('Max', 35);
const person2 = new Person('Anna', 32);
```

### Accessing Properties & Methods

```javascript
console.log(person1.age);   // 35
person2.greet();            // "Hi, I am Anna"
```

---

## Classes in Angular

When using Angular, you'll often define classes which are **never instantiated by you**.

* For example, **components** are created as classes (blueprints for custom HTML elements).
* Angular is the one that instantiates these classes — you never call:

```javascript
new SomeComponent();
```

---

## TypeScript Enhancements to Classes

Angular uses **TypeScript**, which gives you extra features on top of plain JavaScript classes.

### 1. Decorators

```typescript
@Component({})
class SomeComponent {}
```

* Decorators like `@Component` are used by Angular to add **metadata & configuration** to classes.

---

### 2. Access Modifiers

TypeScript allows you to control property & method accessibility:

* `public` (default): Accessible everywhere
* `private`: Accessible only inside the class
* `protected`: Accessible inside the class and subclasses

This helps structure and secure your code better.

---

## Key Takeaways

* Classes are **blueprints for objects**.
* Angular defines many classes (components, services, etc.) that **you don’t instantiate manually**.
* TypeScript provides enhancements:

  * **Decorators** (`@Component`, etc.)
  * **Access modifiers** (`public`, `private`, `protected`)
* You don’t need to dive deeply into classes now — you’ll see them in action throughout the course.

---
