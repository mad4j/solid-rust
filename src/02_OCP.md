# Open/Close Principle (OCP)

The **Open/Closed Principle (OCP)** states that:

> *Software entities (such as classes, modules, and functions) should be **open for extension but closed for modification**.*

This means we should design our code in a way that allows new functionality to be added **without altering existing code**.  

In Rust, we can achieve this by leveraging **traits and generics**, instead of modifying existing structures directly.  

## A Bad Example: A Basic Shape Renderer

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

This implementation **violates OCP** because every time we introduce a new shape (e.g. a Triangle), we must **modify** the `Renderer`'s `render` method.  

## A Better Example: Applying OCP

A better approach is to use **traits** to define a contract for rendering and 

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

Let's build a **pluggable rendering engine** in Rust that follows the **Open/Closed Principle (OCP)** using **traits, dynamic dispatch (`dyn Trait`), and functional composition (`Box<dyn Fn()>`)**.  

---

# **🚀 Pluggable Rendering Engine in Rust**

A **rendering engine** should allow different output backends (e.g., console, OpenGL, Web, file) **without modifying existing code** when adding new renderers.  

## **1. Design Goals**
✅ Follow the **Open/Closed Principle** (new renderers without modifying core logic)  
✅ Use **dynamic dispatch** for extensibility  
✅ Support multiple rendering strategies **at runtime**  
✅ Allow functional-style composition with closures  

---

## **2. Implementing a Pluggable Renderer**

### **Step 1: Define the `Renderer` Trait**
We'll create a `Renderer` trait that all renderers must implement.

```rust
trait Renderer {
    fn render(&self, content: &str);
}
```

---

### **Step 2: Implement Concrete Renderers**
We'll implement **ConsoleRenderer**, **FileRenderer**, and **WebRenderer**.

```rust
use std::fs::File;
use std::io::Write;
use std::rc::Rc;

// Console Renderer
struct ConsoleRenderer;
impl Renderer for ConsoleRenderer {
    fn render(&self, content: &str) {
        println!("Console Output: {}", content);
    }
}

// File Renderer
struct FileRenderer {
    filename: String,
}

impl FileRenderer {
    fn new(filename: &str) -> Self {
        Self {
            filename: filename.to_string(),
        }
    }
}

impl Renderer for FileRenderer {
    fn render(&self, content: &str) {
        let mut file = File::create(&self.filename).expect("Failed to create file");
        writeln!(file, "{}", content).expect("Failed to write to file");
        println!("File Output: Written to {}", self.filename);
    }
}

// Web Renderer (simulated)
struct WebRenderer;
impl Renderer for WebRenderer {
    fn render(&self, content: &str) {
        println!("Web Output: Rendering HTML content -> {}", content);
    }
}
```

---

### **Step 3: Create the Rendering Engine**
The **Rendering Engine** will manage multiple renderers.

```rust
struct RenderingEngine {
    renderers: Vec<Rc<dyn Renderer>>,
}

impl RenderingEngine {
    fn new() -> Self {
        Self {
            renderers: Vec::new(),
        }
    }

    fn add_renderer(&mut self, renderer: Rc<dyn Renderer>) {
        self.renderers.push(renderer);
    }

    fn render_content(&self, content: &str) {
        for renderer in &self.renderers {
            renderer.render(content);
        }
    }
}
```

---

### **Step 4: Functional Composition (Dynamic Strategies)**
For extra flexibility, let's allow **on-the-fly renderers** using **closures** (`Box<dyn Fn()>`).

```rust
struct DynamicRenderer {
    render_fn: Box<dyn Fn(&str)>,
}

impl DynamicRenderer {
    fn new<F: Fn(&str) + 'static>(func: F) -> Self {
        Self {
            render_fn: Box::new(func),
        }
    }
}

impl Renderer for DynamicRenderer {
    fn render(&self, content: &str) {
        (self.render_fn)(content);
    }
}
```

---

### **Step 5: Putting It All Together**
Let's create an **engine, add multiple renderers, and render content dynamically**.

```rust
fn main() {
    let mut engine = RenderingEngine::new();

    // Add predefined renderers
    let console_renderer = Rc::new(ConsoleRenderer);
    let file_renderer = Rc::new(FileRenderer::new("output.txt"));
    let web_renderer = Rc::new(WebRenderer);

    engine.add_renderer(console_renderer.clone());
    engine.add_renderer(file_renderer.clone());
    engine.add_renderer(web_renderer.clone());

    // Add a dynamic renderer using a closure
    let custom_renderer = Rc::new(DynamicRenderer::new(|content| {
        println!("Custom Logger: {}", content);
    }));

    engine.add_renderer(custom_renderer.clone());

    // Render content using all registered renderers
    engine.render_content("Hello, Rust Rendering Engine!");
}
```

---

## **3. Why This Approach Works**
✅ **Open for Extension** – New renderers (OpenGL, Vulkan, etc.) can be added **without modifying** existing code.  
✅ **Closed for Modification** – `RenderingEngine` stays **unchanged** when new rendering backends are introduced.  
✅ **Supports Dynamic Composition** – Custom renderers can be defined **at runtime** using closures.  
✅ **Lightweight and Efficient** – No unnecessary object overhead, leveraging Rust’s **trait objects** and **closures**.

---

## **4. Possible Enhancements**
🚀 Add an **OpenGLRenderer** or **VulkanRenderer** for actual GPU rendering.  
🚀 Implement **asynchronous rendering** with **Tokio** for better performance.  
🚀 Use **Serde** for configuration-based renderer selection.  

---

## **5. Final Thoughts**
This **pluggable rendering engine** in Rust follows the **Open/Closed Principle** by making it easy to add new renderers **without modifying the core engine**.  

Would you like to expand this with **async rendering** or **configurable plugins**? 🚀

Let's enhance our **pluggable rendering engine** with **asynchronous rendering** using **Tokio**. This will allow multiple renderers to work concurrently, improving performance when rendering large amounts of content.

---

# **🚀 Asynchronous Pluggable Rendering Engine in Rust**
By leveraging **async/await** and **Tokio**, we can:
✅ **Render content in parallel** across multiple renderers  
✅ **Improve performance** by avoiding blocking operations (e.g., file I/O, network rendering)  
✅ **Keep the system modular and extensible**  

---

## **1. Updating the Renderer Trait to Support Async**
We modify the `Renderer` trait to return a `Future` using `async fn`.

```rust
use async_trait::async_trait;
use std::fs::File;
use std::io::Write;
use std::sync::Arc;
use tokio::sync::Mutex;

// Define an asynchronous Renderer trait
#[async_trait]
trait Renderer: Send + Sync {
    async fn render(&self, content: &str);
}
```

---

## **2. Implementing Async Renderers**
Each renderer now implements `async fn render`.

### **Console Renderer (Non-blocking)**
```rust
struct ConsoleRenderer;

#[async_trait]
impl Renderer for ConsoleRenderer {
    async fn render(&self, content: &str) {
        println!("Console Output: {}", content);
    }
}
```

### **File Renderer (Async File I/O)**
Instead of blocking `std::fs::File`, we use **Tokio's async file operations**.
```rust
use tokio::fs::File;
use tokio::io::AsyncWriteExt;

struct FileRenderer {
    filename: String,
}

impl FileRenderer {
    fn new(filename: &str) -> Self {
        Self {
            filename: filename.to_string(),
        }
    }
}

#[async_trait]
impl Renderer for FileRenderer {
    async fn render(&self, content: &str) {
        let mut file = File::create(&self.filename)
            .await
            .expect("Failed to create file");
        file.write_all(content.as_bytes())
            .await
            .expect("Failed to write to file");

        println!("File Output: Written to {}", self.filename);
    }
}
```

### **Web Renderer (Simulated Async Request)**
Simulating an async web rendering process using `tokio::time::sleep`.

```rust
use tokio::time::{sleep, Duration};

struct WebRenderer;

#[async_trait]
impl Renderer for WebRenderer {
    async fn render(&self, content: &str) {
        println!("Web Rendering: Preparing...");
        sleep(Duration::from_secs(2)).await; // Simulate async processing
        println!("Web Output: Rendering HTML -> {}", content);
    }
}
```

---

## **3. Creating an Async Rendering Engine**
We use **Arc<Mutex<_>>** to manage shared async state.

```rust
struct RenderingEngine {
    renderers: Vec<Arc<dyn Renderer>>,
}

impl RenderingEngine {
    fn new() -> Self {
        Self {
            renderers: Vec::new(),
        }
    }

    fn add_renderer(&mut self, renderer: Arc<dyn Renderer>) {
        self.renderers.push(renderer);
    }

    async fn render_content(&self, content: &str) {
        let mut tasks = Vec::new();

        for renderer in &self.renderers {
            let renderer = Arc::clone(renderer);
            let content = content.to_string();
            
            // Spawn each rendering task asynchronously
            let task = tokio::spawn(async move {
                renderer.render(&content).await;
            });

            tasks.push(task);
        }

        // Await all rendering tasks concurrently
        futures::future::join_all(tasks).await;
    }
}
```

---

## **4. Running the Async Engine**
We use `tokio::main` to enable async execution.

```rust
#[tokio::main]
async fn main() {
    let mut engine = RenderingEngine::new();

    // Add renderers
    let console_renderer = Arc::new(ConsoleRenderer);
    let file_renderer = Arc::new(FileRenderer::new("output_async.txt"));
    let web_renderer = Arc::new(WebRenderer);

    engine.add_renderer(console_renderer.clone());
    engine.add_renderer(file_renderer.clone());
    engine.add_renderer(web_renderer.clone());

    // Render content asynchronously
    engine.render_content("Hello, Async Rendering Engine!").await;
}
```

---

## **🚀 Why This Works**
✅ **Fully async & non-blocking** – Renders multiple outputs **concurrently**  
✅ **Faster performance** – File I/O and web rendering don’t block execution  
✅ **Open for Extension** – New renderers (like OpenGL, network-based, etc.) can be added **without modifying** existing code  
✅ **Safe concurrent execution** – `Arc` and `Mutex` ensure thread-safe access  

---

## **5. Next Steps**
🚀 **Add an OpenGLRenderer** using `wgpu` for real-time GPU rendering  
🚀 **Implement a network-based renderer** (e.g., send content to a remote server)  
🚀 **Add caching mechanisms** to avoid redundant re-renders  

Would you like to expand this with a **real-time WebSocket renderer** or a **GPU-based renderer**? 🚀

Let's enhance our **async pluggable rendering engine** by adding a **middleware system**. Middleware will allow us to preprocess or modify content before it gets rendered, similar to how middleware in web frameworks (like Express.js or Actix) works.

---

# **🚀 Adding Middleware to an Async Rendering Engine**
## **1. What is Middleware?**
Middleware is a **processing layer** that modifies or logs data before it reaches the final renderer. This enables:  
✅ **Logging and analytics** (e.g., track rendered content)  
✅ **Compression and encryption** (e.g., compress output for web renderers)  
✅ **Security filters** (e.g., sanitize HTML output)  
✅ **Content transformation** (e.g., Markdown-to-HTML conversion)

---

## **2. Implementing Middleware**
We’ll define a **Middleware trait** that allows us to process content **before it gets passed to renderers**.

```rust
use async_trait::async_trait;

#[async_trait]
trait Middleware: Send + Sync {
    async fn process(&self, content: String) -> String;
}
```

---

### **3. Creating Some Middleware Implementations**
#### **(a) Logging Middleware**
This middleware logs the content before rendering.

```rust
struct LoggingMiddleware;

#[async_trait]
impl Middleware for LoggingMiddleware {
    async fn process(&self, content: String) -> String {
        println!("[LOG]: Rendering content -> {}", content);
        content // Pass content unchanged
    }
}
```

#### **(b) Uppercase Middleware**
This middleware converts the content to uppercase.

```rust
struct UppercaseMiddleware;

#[async_trait]
impl Middleware for UppercaseMiddleware {
    async fn process(&self, content: String) -> String {
        content.to_uppercase()
    }
}
```

#### **(c) Markdown-to-HTML Middleware**
This middleware converts **Markdown to HTML** (simulated here, but could use a real Markdown parser).

```rust
struct MarkdownMiddleware;

#[async_trait]
impl Middleware for MarkdownMiddleware {
    async fn process(&self, content: String) -> String {
        if content.starts_with("# ") {
            format!("<h1>{}</h1>", &content[2..])
        } else {
            format!("<p>{}</p>", content)
        }
    }
}
```

---

## **4. Updating the Rendering Engine**
Now, we modify `RenderingEngine` to support **middleware processing** before rendering.

```rust
use std::sync::Arc;
use tokio::sync::Mutex;
use futures::future::join_all;

// Renderer Trait (Async)
#[async_trait]
trait Renderer: Send + Sync {
    async fn render(&self, content: &str);
}

// Rendering Engine
struct RenderingEngine {
    renderers: Vec<Arc<dyn Renderer>>,
    middleware: Vec<Arc<dyn Middleware>>,
}

impl RenderingEngine {
    fn new() -> Self {
        Self {
            renderers: Vec::new(),
            middleware: Vec::new(),
        }
    }

    fn add_renderer(&mut self, renderer: Arc<dyn Renderer>) {
        self.renderers.push(renderer);
    }

    fn add_middleware(&mut self, mw: Arc<dyn Middleware>) {
        self.middleware.push(mw);
    }

    async fn process_content(&self, content: String) -> String {
        let mut processed_content = content;
        
        for mw in &self.middleware {
            processed_content = mw.process(processed_content).await;
        }
        
        processed_content
    }

    async fn render_content(&self, content: &str) {
        let processed_content = self.process_content(content.to_string()).await;
        let mut tasks = Vec::new();

        for renderer in &self.renderers {
            let renderer = Arc::clone(renderer);
            let content_clone = processed_content.clone();

            let task = tokio::spawn(async move {
                renderer.render(&content_clone).await;
            });

            tasks.push(task);
        }

        join_all(tasks).await;
    }
}
```

---

## **5. Updating the Renderers**
Our async renderers remain unchanged from the previous implementation.

```rust
struct ConsoleRenderer;

#[async_trait]
impl Renderer for ConsoleRenderer {
    async fn render(&self, content: &str) {
        println!("Console Output: {}", content);
    }
}
```

```rust
struct FileRenderer {
    filename: String,
}

impl FileRenderer {
    fn new(filename: &str) -> Self {
        Self {
            filename: filename.to_string(),
        }
    }
}

#[async_trait]
impl Renderer for FileRenderer {
    async fn render(&self, content: &str) {
        let mut file = tokio::fs::File::create(&self.filename)
            .await
            .expect("Failed to create file");
        tokio::io::AsyncWriteExt::write_all(&mut file, content.as_bytes())
            .await
            .expect("Failed to write to file");

        println!("File Output: Written to {}", self.filename);
    }
}
```

---

## **6. Running Everything**
Now, we create the **engine, add middleware, add renderers, and render content**.

```rust
#[tokio::main]
async fn main() {
    let mut engine = RenderingEngine::new();

    // Add middleware
    let log_middleware = Arc::new(LoggingMiddleware);
    let uppercase_middleware = Arc::new(UppercaseMiddleware);
    let markdown_middleware = Arc::new(MarkdownMiddleware);

    engine.add_middleware(log_middleware.clone());
    engine.add_middleware(uppercase_middleware.clone());
    engine.add_middleware(markdown_middleware.clone());

    // Add renderers
    let console_renderer = Arc::new(ConsoleRenderer);
    let file_renderer = Arc::new(FileRenderer::new("output_middleware.txt"));

    engine.add_renderer(console_renderer.clone());
    engine.add_renderer(file_renderer.clone());

    // Render content with middleware processing
    engine.render_content("# Hello, Rust Middleware!").await;
}
```

---

## **7. Expected Output**
```
[LOG]: Rendering content -> # Hello, Rust Middleware!
Console Output: <H1>HELLO, RUST MIDDLEWARE!</H1>
File Output: Written to output_middleware.txt
```
- **Logging Middleware** prints the original content.
- **Uppercase Middleware** converts it to uppercase.
- **Markdown Middleware** transforms it into an `<h1>` tag.
- **Renderers** output the final transformed content.

---

## **🚀 Why This Approach Works**
✅ **Open for Extension** – New middleware (e.g., sanitization, encryption) can be added **without modifying** existing logic.  
✅ **Closed for Modification** – The core engine remains **unchanged** when extending functionality.  
✅ **Asynchronous & Efficient** – Middleware and rendering operations run **concurrently** using async/await.  
✅ **Decoupled & Modular** – Middleware is completely independent from the rendering process.

---

## **8. Next Steps**
🚀 **Add a WebSocket renderer** to stream content live.  
🚀 **Create an API middleware** to fetch external data before rendering.  
🚀 **Implement a caching middleware** to prevent redundant rendering.  

Would you like an example with **real-time WebSocket rendering**? 🚀

Let's enhance our **async pluggable rendering engine** with **real-time WebSocket rendering** using **Tokio and Warp**. This will allow us to dynamically push rendered content to connected WebSocket clients.  

---

# **🚀 Real-Time WebSocket Rendering in Rust**  

## **1. Why WebSockets?**
WebSockets enable **real-time, bidirectional communication**, making them ideal for:  
✅ **Live dashboards** (e.g., stock prices, system logs, game scores)  
✅ **Streaming updates** (e.g., collaborative editing, notifications)  
✅ **Efficient real-time rendering** without polling  

---

## **2. Setting Up WebSocket Rendering**
We will:  
1. **Create a WebSocket server** using `warp`  
2. **Store connected clients** in a shared state (`Arc<Mutex<Vec<Tx>>>`)  
3. **Broadcast rendered content** in real-time  

---

### **3. Install Dependencies**  
We need `tokio`, `async-trait`, `warp`, and `futures`. Add these to `Cargo.toml`:  

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
warp = "0.3"
async-trait = "0.1"
futures = "0.3"
```

---

## **4. Implementing the WebSocket Renderer**  

### **(a) Shared State for WebSocket Clients**  
We use `Arc<Mutex<Vec<Tx>>>` to track all active WebSocket connections.  

```rust
use std::sync::Arc;
use tokio::sync::Mutex;
use warp::ws::{Message, WebSocket};
use warp::Filter;
use futures::{SinkExt, StreamExt};

type Tx = tokio::sync::mpsc::UnboundedSender<String>;
type Clients = Arc<Mutex<Vec<Tx>>>;
```

---

### **(b) WebSocket Renderer**
This renderer **broadcasts** rendered content to all connected WebSocket clients.

```rust
use async_trait::async_trait;

struct WebSocketRenderer {
    clients: Clients,
}

impl WebSocketRenderer {
    fn new(clients: Clients) -> Self {
        Self { clients }
    }

    async fn broadcast(&self, content: &str) {
        let mut clients = self.clients.lock().await;
        clients.retain(|tx| tx.send(content.to_string()).is_ok()); // Remove closed connections
    }
}

#[async_trait]
impl Renderer for WebSocketRenderer {
    async fn render(&self, content: &str) {
        println!("WebSocket Output: Broadcasting -> {}", content);
        self.broadcast(content).await;
    }
}
```

---

### **(c) WebSocket Server Handler**
This function **accepts new WebSocket clients** and adds them to the `clients` list.

```rust
async fn websocket_handler(ws: WebSocket, clients: Clients) {
    let (mut sender, mut receiver) = ws.split();
    let (tx, mut rx) = tokio::sync::mpsc::unbounded_channel();
    
    {
        let mut clients_locked = clients.lock().await;
        clients_locked.push(tx);
    }
    
    println!("New WebSocket client connected!");

    tokio::spawn(async move {
        while let Some(Ok(msg)) = rx.recv().await {
            sender.send(Message::text(msg)).await.ok();
        }
    });

    while let Some(Ok(_)) = receiver.next().await {} // Keep connection open
}
```

---

### **(d) Launching the WebSocket Server**
We use `warp` to start an HTTP server with a **WebSocket route**.

```rust
async fn start_websocket_server(clients: Clients) {
    let clients_filter = warp::any().map(move || clients.clone());

    let ws_route = warp::path("ws")
        .and(warp::ws())
        .and(clients_filter)
        .map(|ws: warp::ws::Ws, clients| {
            ws.on_upgrade(move |socket| websocket_handler(socket, clients))
        });

    warp::serve(ws_route).run(([127, 0, 0, 1], 3030)).await;
}
```

---

## **5. Integrating with the Rendering Engine**  
Now we update our `main` function to:  
1. **Start the WebSocket server** in a background task  
2. **Add WebSocketRenderer** to the engine  

```rust
#[tokio::main]
async fn main() {
    let clients: Clients = Arc::new(Mutex::new(Vec::new()));
    
    // Start WebSocket server in a separate task
    let ws_clients = clients.clone();
    tokio::spawn(async move {
        start_websocket_server(ws_clients).await;
    });

    let mut engine = RenderingEngine::new();

    // Add renderers
    let console_renderer = Arc::new(ConsoleRenderer);
    let file_renderer = Arc::new(FileRenderer::new("output_realtime.txt"));
    let websocket_renderer = Arc::new(WebSocketRenderer::new(clients));

    engine.add_renderer(console_renderer);
    engine.add_renderer(file_renderer);
    engine.add_renderer(websocket_renderer);

    // Render content (pushed to WebSockets in real-time)
    engine.render_content("Live Update: Hello, WebSockets!").await;
}
```

---

## **6. Running the WebSocket Server**
1. Start the Rust program:
   ```
   cargo run
   ```
   The server will run at `ws://localhost:3030/ws`.

2. Connect via a WebSocket client (browser/CLI):
   ```javascript
   const ws = new WebSocket("ws://localhost:3030/ws");
   ws.onmessage = (event) => console.log("Received:", event.data);
   ```

3. Every time `render_content` is called, all **WebSocket clients** receive updates in real-time!

---

## **7. Expected Output**
### **Console Output (Server)**
```
New WebSocket client connected!
WebSocket Output: Broadcasting -> Live Update: Hello, WebSockets!
Console Output: Live Update: Hello, WebSockets!
File Output: Written to output_realtime.txt
```

### **WebSocket Client Output (Browser)**
```
Received: Live Update: Hello, WebSockets!
```

---

## **8. Why This Works**
✅ **Real-time updates** – Clients receive new content **instantly**  
✅ **Efficient broadcasting** – Uses an **async event-driven** model  
✅ **Scalable design** – Supports multiple clients  
✅ **Decoupled system** – Middleware, renderers, and WebSocket server are **independent**  

---

## **9. Next Steps**
🚀 **Add an OpenGL renderer** for live graphics rendering  
🚀 **Use Redis pub/sub** for distributed real-time rendering  
🚀 **Implement WebSocket authentication**  

Would you like to extend this with **live updates from a database**? 🚀

