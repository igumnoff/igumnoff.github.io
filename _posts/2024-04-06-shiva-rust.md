---
layout: post
title:  "Shiva — Open Source project in Rust for parsing and generating documents of any type"
date:   2024-04-06 19:30:59 +0500
categories: rust, parser, pdf
---
The idea for the project came to me while working on a document search engine project. There is such a library
like Apache Tika, written in Java, which can parse documents of various types.
For my search engine to work, it must be able to extract text from documents
different types (PDF, DOC, XLS, HTML, XML, JSON, etc.). I wrote the search engine itself in Rust.
But, unfortunately, in the Rust world there is no library that can parse documents of all types.


For this reason, I had to use Apache Tika and call it from my Rust code. What are the disadvantages of such a solution?

1. It is necessary to install Java on each computer where my search engine will be launched.

2. Very high RAM requirements. Apache Tika uses a lot of memory. Because Java has a garbage collector that is not very efficient, it has to allocate a lot of memory to the JVM.

At that time, I did not write a library in Rust that can parse documents of all types,
because the employer was not ready to pay for several years of my development. But the idea remained
in my head. And so, I decided to take this step and began writing my own library in Rust.


Only I decided to make not just a parser, but also a document generator.
Because it will be more useful and versatile. This is how it is done in the well-known Pandoc library.

In fact, the task of creating such a library is quite interesting. We need to develop a good
architecture of the core library, so that later it would be easy to add new parsers and
generators for various types of documents. I chose a usage based approach
Common Document Model (CDM). That is, the code of any parser must convert the document into CDM,
and any generator code must convert the CDM into a document.

In versions 0.1.x I used dyn and downcast. In the core module I laid out the following structures:

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

Since Rust doesn't have Reflection, I decided to use Any. This way I can store different types of document elements in one vector. If necessary, I can cast them to the desired type via downcast_ref and downcast_mut. To do this, I added methods to the Element trait for all types of elements (for example, paragraph_as_ref, paragraph_as_mut, etc.).

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

In version 0.2.x I refactored the code. Switched from dyn and downcast to enum and pattern matching. And the core module began to look like this:

```rust
#[derive(Debug, Clone)]
pub enum Element {
    Text {
        text: String,
        size: u8,
    },
    Header {
        level: u8,
        text: String,
    },
    Paragraph {
        elements: Vec<Element>,
    },
    Table {
        headers: Vec<TableHeader>,
        rows: Vec<TableRow>,
    },
    List {
        elements: Vec<ListItem>,
        numbered: bool,
    },
    Image {
        bytes: Bytes,
        title: String,
        alt: String,
        image_type: ImageType,
    },
    Hyperlink {
        title: String,
        url: String,
        alt: String,
    },
}

#[derive(Debug, Clone)]
pub struct ListItem {
    pub element: Box<Element>,
}
#[derive(Debug, Clone)]
pub struct TableHeader {
    pub element: Box<Element>,
    pub width: f32,
}

#[derive(Debug, Clone)]
pub struct TableRow {
    pub cells: Vec<TableCell>,
}

#[derive(Debug, Clone)]
pub struct TableCell {
    pub element: Box<Element>,
}

#[derive(Debug, Clone)]
pub enum ImageType {
    Png,
    Jpeg,
}

```

In the future I plan to do a refactoring for Document. Transfer indents from it and add the Page entity.

The following parsers and generators are currently implemented:

- Plain text
- Markdown
- HTML
- PDF

To connect my library to your project, you need to add the following line to Cargo.toml:


```rust
[dependencies]
shiva = {  version = "0.2.0", features = ["html", "markdown", "text", "pdf"] }
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

The current status of the project is MVP (Minimum Viable Product). I plan to add support for all document types,
that Apache Tika supports over the next few years. And I’ll add more in the near future
deep support for PDF documents, since PDF is the most popular document type. I'll also add
mode of operation as a web service so that I can use my library through the REST API, and not just through the CLI.

The project's source code is on [GitHub](https://github.com/igumnoff/shiva). 
