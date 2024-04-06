---
layout: post
title:  "Shiva — Open Source project in Rust for parsing and generating documents of any type"
date:   2024-04-06 19:30:59 +0500
categories: rust, parser, pdf
---
The idea for the project came to me while working on a document search engine project. There is a library such as Apache Tika, written in Java, which can parse documents of various types. For my search engine to work, it must be able to extract text from different types of documents (PDF, DOC, XLS, HTML, XML, JSON, etc.). I wrote the search engine itself in Rust. But, unfortunately, in the Rust world there is no library that can parse documents of all types.


For this reason, I had to use Apache Tika and call it from my Rust code. What are the disadvantages of such a solution?

1. It is necessary to install Java on each computer where my search engine will be launched.

2. Very high RAM requirements. Apache Tika uses a lot of memory. Due to the fact that Java has a garbage collector that is not very efficient, it has to allocate a lot of memory to the JVM.

At that time, I did not write a library in Rust that could parse documents of all types, because the employer was not ready to pay for several years of my development. But the idea remained in my head. And so, I decided to take this step and began writing my own library in Rust.

In fact, the task of creating such a library is quite interesting. It is necessary to develop a good library core architecture so that later it is easy to add new parsers and generators for various types of documents. I chose an approach based on using the Common Document Model (CDM). That is, any parser code must convert a document into a CDM, and any generator code must convert a CDM into a document.

In the core module I laid out the following structures:

```rust
pub struct Document {
    pub elements: Vec<Box<dyn Element>>,
    pub page_width: f32,
    pub page_height: f32,
    pub left_page_indent: f32,
    pub right_page_indent: f32,
    pub top_page_indent: f32,
    pub bottom_page_indent: f32,
    pub page_header: Vec<Box<dyn Element>>,
    pub page_footer: Vec<Box<dyn Element>>,
}

trait Element  {
   fn as_any(&self) -> &dyn Any; 

   fn as_any_mut(&mut self) -> &mut dyn Any;

   fn element_type(&self) -> ElementType;
}

pub enum ElementType {
   Text,
   Paragraph,
   Image,
   Hyperlink,
   Header,
   Table,
   TableHeader,
   TableRow,
   TableCell,
   List,
   ListItem,
   PageBreak,
   TableOfContents,
}
```
Since Rust doesn’t have Reflection, I decided to use Any. This way I can store different types of document elements in one vector. If necessary, I can cast them to the desired type via downcast_ref and downcast_mut. To do this, I added methods to the Element trait for all types of elements (for example, paragraph_as_ref, paragraph_as_mut, etc.)
```rust
    fn paragraph_as_ref(&self) -> anyhow::Result<&ParagraphElement> {
        Ok(self
            .as_any()
            .downcast_ref::<ParagraphElement>()
            .ok_or(CastingError::Common)?)
    }

    fn paragraph_as_mut(&mut self) -> anyhow::Result<&mut ParagraphElement> {
        Ok(self
            .as_any_mut()
            .downcast_mut::<ParagraphElement>()
            .ok_or(CastingError::Common)?)
    }
```
In order to add a new document type, it is enough to implement the TransformerTrait trait:

```rust
pub trait TransformerTrait {
    fn parse(document: &Bytes, images: &HashMap<String, Bytes>) -> anyhow::Result<Document>;
    fn generate(document: &Document) -> anyhow::Result<(Bytes, HashMap<String, Bytes>)>;
}
```
The following parsers and generators are currently implemented:


- Plain text
- Markdown
- HTML
- PDF

To connect my library to your project, you need to add the following line to Cargo.toml:

```rust
[dependencies]
shiva = "0.1.14"
```
Example of using the library:

```rust
fn main() {
    let input_vec = std::fs::read("input.html").unwrap();
    let input_bytes = bytes::Bytes::from(input_vec);
    let document = shiva::html::Transformer::parse(&input_bytes, &HashMap::new()).unwrap();
    let output_bytes = shiva::markdown::Transformer::generate(&document, &HashMap::new()).unwrap();
    std::fs::write("out.md", output_bytes).unwrap();
}
```
The current status of the project is MVP (Minimum Viable Product). I plan to add support for all document types that Apache Tika supports over the next few years. And in the near future I will add deeper support for PDF documents, since PDF is the most popular type of document. I will also add a web service mode so that you can use my library via the REST API, and not just via the CLI.

