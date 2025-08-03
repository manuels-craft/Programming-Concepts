# Stop Misusing Arrow Functions. Here's What You Need to Know

After reviewing hundreds of pull requests, exploring complex codebases, and debugging JavaScript issues with colleagues, I've seen the same pattern emerge repeatedly. Developers are using arrow functions and traditional functions interchangeably without understanding the fundamental differences.

## The Real Problem
Most developers can write the syntax perfectly, but they don't grasp **why arrow functions were introduced to JavaScript** or **what specific problems they solve**.

### The Root Cause
The issue isn't a lack of technical ability. It's how arrow functions are typically taught. Most tutorials simply:

- Show the syntax differences  
- Mention that arrow functions are "shorter"  
- Move on to the next topic 

What's missing is **real-world context** and **the decision-making process of when to use each type**.  
As a result, developers memorize syntax but fail to understand the reasoning behind choosing one function type over another. This leads to arbitrary decisions that cause subtle bugs, unreadable code, and missed optimization opportunities.

Even worse, many developers overlook **hidden features built into different function types**, including powerful debugging techniques and rarely discussed capabilities that can save hours of troubleshooting.  

---

## What I've Observed
In nearly every code review and codebase exploration, I've seen developers struggle with at least one of these concepts:
- When `this` binding matters (and when it doesn't)  
- How function hoisting affects code organization  
- Why arrow functions can't be constructors  
- When implicit returns improve vs. hurt readability  
- Performance implications in different contexts  

These issues aren’t just theoretical. I've seen **production bugs** and **countless hours wasted debugging** problems that could have been prevented with better function choices.


Before We Compare Arrow and Traditional Functions
Before jumping into the differences between arrow functions and traditional functions, it's important to understand a few core concepts first.

The concepts we need to cover are:
- this keyword
- constructors and the new keyword
- prototypes
- function scope and context binding

Once we understand what each of these concepts is, we will look at how functions use them in conjunction and why arrow and traditional functions behave differently.


## The `this` Keyword.
### What this Is
Think of `this` as the “name tag” you wear at the coffee shop. It tells everyone who you are in the context of your current job. The name tag changes depending on which counter or section you're working at. In JavaScript, `this` works the same way: it always refers to the object that is “wearing the name tag” when the function is called.
```
const barista = {
  name: "Manny",
  greetCustomer() {
    console.log(`Hi, I'm ${this.name}, and I'll take your order!`);
  }
};

barista.greetCustomer(); // Hi, I'm Manny, ...
```
In this example, we have an object called `barista` with a property `name` and a method `greetCustomer`. Inside the method, we use `this.name` to access the name of the barista. When we call `barista.greetCustomer()`, the object `barista` is the one calling the method, so `this` refers to `barista`. As a result, the message prints “Hi, I'm Manny, and I'll take your order!” just like a barista wearing their name tag at the counter.

Here's your improved explanation clearly structured:

---

### Why Developers Complain About It (and Why It’s Just Lack of Knowledge)

Many developers complain that `this` is confusing or unpredictable. That’s like a new barista saying, “Why does my name tag keep changing depending on where I’m standing?” The truth is, `this` follows consistent rules. You just need to understand how JavaScript decides who’s wearing the name tag at any moment.

Let's think about two scenarios in a workplace to clarify this further:

#### * JS Non-strict mode (Relaxed Workplace)

Imagine Manny, a barista, receives a personal text message and steps away from his counter to the back of the coffee shop to reply. While Manny is away, another employee(let's call him "Global") steps in and starts handling Manny's responsibilities. Manny's absence means someone else (the global object) casually takes over his job, leading to unexpected results.

In JavaScript (without strict mode), the same thing happens. When a function is detached from its original object and invoked without context, JavaScript defaults to the global object, causing potential confusion:

```js
// Non-strict mode example (browser environment)
window.name = "Global Manny";

const barista = {
  name: "Manny",
  greetCustomer() {
    console.log(`Hi, I'm ${this.name}`);
  }
};

const greet = barista.greetCustomer;
greet(); // Hi, I'm Global Manny
```

Here, the context (`this`) unintentionally points to the global object, causing potentially confusing outcomes.

#### * JS Strict mode (Manager Present)
Now imagine a strict workplace scenario, like when your boss or manager is around. If someone’s personal phone receives a text, no one would casually respond. Everyone respects strict rules and roles. Thus, if there's no clearly defined recipient, the message goes unanswered, immediately revealing an issue.

In JavaScript strict mode, something similar occurs. If a function loses its original context, `this` becomes explicitly `undefined`. No casual fallback happens, making problems clearer and easier to catch:

```js
"use strict";

const barista = {
  name: "Manny",
  greetCustomer() {
    console.log(`Hi, I'm ${this.name}`);
  }
};

const greet = barista.greetCustomer;
greet(); // TypeError or "Hi, I'm undefined"
```

With strict mode, JavaScript clearly indicates there's an issue rather than silently using the global context. This clarity helps developers avoid bugs and confusion.

By understanding these scenarios clearly, you'll see why `this` behaves consistently—even if it initially feels unpredictable.

#### Why this Is Useful
---
Imagine if in the coffee shop you couldn’t tell which barista was making a drink. Chaos. this lets functions know which object they are currently working with.

```
function makeDrink(drink) {
  console.log(`${this.name} made a ${drink}`);
}

const barista1 = { name: "Manny" };
const barista2 = { name: "Alex" };

makeDrink.call(barista1, "Latte");
makeDrink.call(barista2, "Cappuccino");
```

This code defines a regular function makeDrink that expects a drink name. By using the .call method, we explicitly tell JavaScript what this should be when running the function. The first call sets this to barista1, so it prints “Manny made a Latte.” The second call sets this to barista2, so it prints “Alex made a Cappuccino.” Because this is dynamic, the same function can be reused by different objects without having to rewrite or duplicate code, making it much more flexible.

### * Common Workarounds When Developers Don’t Understand It
When developers don’t fully grasp this, they often resort to workarounds like storing the value in another variable, such as const self = this;, to retain the correct context.

```
const barista = {
  name: "Manny",
  startShift() {
    const self = this;

    setTimeout(function () {
      console.log(`${self.name} is starting the shift`);
    }, 1000);
  }
};

barista.startShift(); // Manny is starting the shift (after 1 second)
```

In this example, we store this in a variable called self because the regular function used inside setTimeout has its own this, which doesn't point to the barista object. By using self.name, we still access the correct object even though the context (this) changed inside the callback.

While this approach works, it’s considered a workaround. A cleaner and better solution would be to use an arrow function instead, because arrow functions do not have their own this, but instead inherit it from the surrounding scope.

Later on, I'll explain in detail why arrow functions don't possess their own this, how traditional functions manage their own this, and exactly where that context comes from.


### * How We Can Find Out What this Refers To
The easiest way to see what this refers to is to log it. Imagine a barista taking off their apron and letting someone else wear it. The name tag changes depending on who is wearing it at the moment.


```
function whoAmI() {
  console.log(this);
}

// Case 1: Called without an owner
whoAmI();

// Case 2: Called as a method of an object
const barista = {
  name: "Manny",
  whoAmI
};

barista.whoAmI();
```

In the first case, we call `whoAmI()` by itself. Since no object is calling it, `this` is `undefined` in strict mode (or the global object in non-strict mode).
In the second case, we create an object `barista` and assign the same function as a method. When we call `barista.whoAmI()`, the function now has an owner. The `this` keyword refers to the `barista` object, so the logged value is that object.

Thinking of the coffee shop analogy, in the first call there’s no barista wearing the name tag, so there’s no name tag to read. In the second call, Manny is wearing the name tag, so this correctly identifies Manny as the one on duty.


### * How Developers Feel After They Decide to Understand It
Once developers understand this, everything feels predictable.

```
const barista = { 
  name: "Manny", 
  greet() { console.log(this.name) } 
};

const greet = barista.greet;
greet(); // undefined

greet.call(barista); // Manny
```

At first, calling greet() without any context prints undefined because there is no object attached to the call. But when we use greet.call(barista), we explicitly set this to barista, so the output becomes “Manny.” Once developers learn that this is entirely determined by how a function is called, it stops feeling unpredictable and instead becomes a powerful tool to write flexible, reusable functions.


<!-- 
### Constructors and the new Keyword
### Prototypes
### Function Scope and Context Binding -->











<!-- ## Why Talk About `this` and Functions Together?

You might wonder why I’m spending so much time on `this` in an article about functions. The reason is simple: **understanding how `this` behaves is one of the biggest differences between traditional and arrow functions.**

Something I’ve never understood is why so many tutorials introduce arrow functions without first explaining how traditional functions work, especially when it comes to the `this` word.  


New developers often don’t need arrow functions when they first start learning JavaScript. During my first job as a front-end developer, I spent about two years using only traditional functions and never ran into any issues.

Looking back, those two years were valuable because they helped me understand and appreciate the problems that arrow functions were designed to solve. By the time I started using arrow functions, I knew why they were useful—not just how to write them.

Building a solid foundation with traditional functions before adopting arrow functions leads to deeper understanding and fewer mistakes in the long run.

When tutorials jump too quickly to new syntax, without enough real-world context or hours of practical experience, it often leads to confusion and misuse. That’s why I’ve chosen to connect these concepts properly here, so you understand **why arrow functions exist**, **what problems they solve**, and **when it actually makes sense to use them.** -->
