# Open/Close Principle (OCP)

The **Open/Closed Principle (OCP)** states that:

> *Software entities (such as classes, modules, and functions) should be **open for extension but closed for modification**.*

This means we should design our code in a way that allows new functionality to be added **without altering existing code**.  

In Rust, we can achieve this by leveraging **traits and generics**, instead of modifying existing structures directly.  

## Example: A Basic Shape Renderer

Imagine we have a simple shape rendering system. A naive approach might look like this:  

```rust
struct Circle;
struct Square;

struct Renderer;

impl Renderer {
    fn render(&self, shape: &str) {
        match shape {
            "circle" => println!("Rendering a circle"),
            "square" => println!("Rendering a square"),
            _ => println!("Unknown shape"),
        }
    }
}
```

This implementation **violates OCP** because every time we introduce a new shape (e.g., a Triangle), we must **modify** the `Renderer`'s `render` method.  

## Applying the Open/Closed Principle  

A better approach is to use **traits** to define a contract for rendering:  

```rust
trait Shape {
    fn render(&self);
}

struct Circle;
struct Square;

impl Shape for Circle {
    fn render(&self) {
        println!("Rendering a circle");
    }
}

impl Shape for Square {
    fn render(&self) {
        println!("Rendering a square");
    }
}

struct Renderer;

impl Renderer {
    fn render<T: Shape>(&self, shape: &T) {
        shape.render();
    }
}
```

## Benefits of This Approach  

1. **Open for Extension** – We can add new shapes (e.g., `Triangle`) without modifying existing code.  
2. **Closed for Modification** – The `Renderer` struct remains unchanged regardless of new shapes.  

## Adding a New Shape Without Modification  

```rust
struct Triangle;

impl Shape for Triangle {
    fn render(&self) {
        println!("Rendering a triangle");
    }
}

fn main() {
    let renderer = Renderer;
    let circle = Circle;
    let square = Square;
    let triangle = Triangle;

    renderer.render(&circle);
    renderer.render(&square);
    renderer.render(&triangle); // Works without modifying Renderer!
}
```

This implementation follows the **Open/Closed Principle** by allowing new shapes to be added **without modifying existing code**, keeping our system flexible and maintainable. 🚀

Let's explore a more advanced approach to the **Open/Closed Principle (OCP)** in Rust, using **dynamic dispatch** with trait objects and enums.  

---

## **Advanced OCP in Rust: Dynamic Dispatch and Enums**

In the previous example, we used **generics** and **static dispatch** (`T: Shape`), which works well but can lead to excessive monomorphization (code duplication at compile-time). Instead, we can use **dynamic dispatch** (`dyn Shape`) or **enums** to achieve extensibility.

### **1. Using Dynamic Dispatch (`dyn Shape`)**  

With **dynamic dispatch**, we store different shape types as **trait objects** (`Box<dyn Shape>`), allowing us to handle multiple shapes **without modifying existing code**.

```rust
trait Shape {
    fn render(&self);
}

struct Circle;
struct Square;

impl Shape for Circle {
    fn render(&self) {
        println!("Rendering a circle");
    }
}

impl Shape for Square {
    fn render(&self) {
        println!("Rendering a square");
    }
}

// Renderer handles any Shape dynamically
struct Renderer {
    shapes: Vec<Box<dyn Shape>>,
}

impl Renderer {
    fn new() -> Self {
        Self { shapes: Vec::new() }
    }

    fn add_shape(&mut self, shape: Box<dyn Shape>) {
        self.shapes.push(shape);
    }

    fn render_all(&self) {
        for shape in &self.shapes {
            shape.render();
        }
    }
}

fn main() {
    let mut renderer = Renderer::new();
    
    renderer.add_shape(Box::new(Circle));
    renderer.add_shape(Box::new(Square));
    
    renderer.render_all();
}
```

### **Why This Works?**

✅ **Open for Extension** – We can add new shapes **without modifying** `Renderer`.  
✅ **Closed for Modification** – The `Renderer` struct stays unchanged regardless of new shapes.  

---

### **2. Using Enums for Compile-Time Safety**  

Instead of **dynamic dispatch**, we can use **enums** to enforce shape types at compile time.  

```rust
enum Shape {
    Circle,
    Square,
}

impl Shape {
    fn render(&self) {
        match self {
            Shape::Circle => println!("Rendering a circle"),
            Shape::Square => println!("Rendering a square"),
        }
    }
}

struct Renderer {
    shapes: Vec<Shape>,
}

impl Renderer {
    fn new() -> Self {
        Self { shapes: Vec::new() }
    }

    fn add_shape(&mut self, shape: Shape) {
        self.shapes.push(shape);
    }

    fn render_all(&self) {
        for shape in &self.shapes {
            shape.render();
        }
    }
}

fn main() {
    let mut renderer = Renderer::new();
    
    renderer.add_shape(Shape::Circle);
    renderer.add_shape(Shape::Square);
    
    renderer.render_all();
}
```

### **When to Use Each Approach?**

| Approach | Pros | Cons |
|----------|------|------|
| **Dynamic Dispatch (`dyn Shape`)** | More flexible, allows runtime polymorphism | Slightly slower due to vtable lookup |
| **Enums (`enum Shape`)** | Faster (no dynamic dispatch), safer | Requires modifying enum when adding new shapes |

---

## **Conclusion**  

By using **trait objects** or **enums**, we can design our code to be **open for extension but closed for modification**, adhering to the **Open/Closed Principle** in Rust.  

- Use **`dyn Trait`** for flexibility and runtime polymorphism.  
- Use **enums** for performance and compile-time safety.  

Would you like an even more advanced example, perhaps with **functional-style composition** or **strategy patterns**? 🚀


Let's take the **Open/Closed Principle (OCP)** a step further using **functional-style composition** and the **Strategy Pattern** in Rust. These techniques allow us to extend behavior dynamically while keeping existing code unchanged.  

---

## **🚀 Advanced OCP in Rust: Strategy Pattern & Functional Composition**  

Instead of using **inheritance** (common in OOP languages like Java), Rust encourages **composition** via **traits and function pointers (closures)**. This makes our code more flexible and truly **open for extension but closed for modification**.  

---

## **1. Strategy Pattern in Rust (Using Traits & Box<dyn Fn()>)**

The **Strategy Pattern** allows us to define a family of algorithms (or behaviors) and select them dynamically at runtime.  

### **Example: Dynamic Rendering Strategies**
```rust
use std::rc::Rc;

trait RenderStrategy {
    fn render(&self);
}

// Concrete strategies
struct ConsoleRenderer;
impl RenderStrategy for ConsoleRenderer {
    fn render(&self) {
        println!("Rendering to console");
    }
}

struct WebRenderer;
impl RenderStrategy for WebRenderer {
    fn render(&self) {
        println!("Rendering to web canvas");
    }
}

// Context that delegates rendering
struct Shape {
    name: String,
    render_strategy: Rc<dyn RenderStrategy>,
}

impl Shape {
    fn new(name: &str, render_strategy: Rc<dyn RenderStrategy>) -> Self {
        Self {
            name: name.to_string(),
            render_strategy,
        }
    }

    fn render(&self) {
        print!("Rendering shape: {} -> ", self.name);
        self.render_strategy.render();
    }
}

fn main() {
    let console_renderer = Rc::new(ConsoleRenderer);
    let web_renderer = Rc::new(WebRenderer);

    let circle = Shape::new("Circle", console_renderer.clone());
    let square = Shape::new("Square", web_renderer.clone());

    circle.render(); // Renders to console
    square.render(); // Renders to web canvas
}
```

### **Why This Works?**
✅ **Open for Extension** – We can add new rendering strategies (`FileRenderer`, `OpenGLRenderer`, etc.) **without modifying `Shape`**.  
✅ **Closed for Modification** – The existing rendering logic remains **untouched**.  

---

## **2. Functional Composition with Closures (`Box<dyn Fn()>`)**
For even greater flexibility, we can use **closures** as dynamic strategies instead of traits.  

### **Example: Dynamic Behavior with Closures**
```rust
struct Shape {
    name: String,
    render_fn: Box<dyn Fn()>,
}

impl Shape {
    fn new(name: &str, render_fn: impl Fn() + 'static) -> Self {
        Self {
            name: name.to_string(),
            render_fn: Box::new(render_fn),
        }
    }

    fn render(&self) {
        print!("Rendering shape: {} -> ", self.name);
        (self.render_fn)();
    }
}

fn main() {
    let console_renderer = || println!("Rendering to console");
    let web_renderer = || println!("Rendering to web canvas");

    let circle = Shape::new("Circle", console_renderer);
    let square = Shape::new("Square", web_renderer);

    circle.render();
    square.render();
}
```

### **Why This Works?**
✅ **No Trait Boilerplate** – Just pass any function or closure as a strategy.  
✅ **Extremely Flexible** – Allows on-the-fly customization without defining new structs.  
✅ **Minimal Overhead** – Uses Rust’s powerful **first-class functions** for dynamic behavior.  

---

## **🚀 Summary: OCP in Rust**
| Approach | Pros | Cons |
|----------|------|------|
| **Trait Objects (`dyn Trait`)** | Stronger type safety, extensible | Some overhead due to dynamic dispatch |
| **Enum-Based Strategy** | Faster, avoids dynamic dispatch | Requires modifying enum for new strategies |
| **Functional Composition (`Box<dyn Fn()>`)** | Extremely flexible, no trait boilerplate | Harder to document & enforce contracts |

---
## **🎯 When to Use What?**
- Use **traits (`dyn Trait`)** if your behaviors have a clear structure & should be **reusable**.  
- Use **closures (`Box<dyn Fn()>`)** if you need **lightweight, ad-hoc strategy selection**.  
- Use **enums** if the set of behaviors is **known & limited** at compile-time.  

Would you like a real-world example, such as a **pluggable rendering engine** or a **middleware system**? 🚀