---
layout: post
title:  "Metatron - Open Source library for generating reports in Rust language"
date:   2024-04-19 22:32:01 +0500
categories: rust, parser, pdf
---
A year ago, the idea arose to rewrite the entire Java backend in Rust, which I have been developing and supporting for several years. I found all the analogues of libraries and frameworks from the Java world in the Rust ecosystem:

- Spring Framework = Axum
- Jackson = Serde
- Hibernate = SeaORM
- Bouncy Castle = Rustls


In the world of Rust, I could not find an analogue of only one library - JasperReports, which is used to generate reports. For this reason, we had to abandon the idea of ​​rewriting the entire project in Rust.

Since the niche for generating answers in Rust is empty, I created my own open source project - a report generator.

I decided not to reinvent the wheel, so I took many ideas from JasperReports. For example, to generate a report, you need two things:

1. Report template.
2. Data that needs to be inserted into the report.

I chose a report template in YAML format rather than XML because YAML is easier for humans to read and edit.
By the way, it is for this reason that JasperReports has a WYSIWYG report template editor - it’s quite difficult to navigate XML.

Example report template:
```yaml
title:
  - header: $P{company_name} Employee Report
    level: 1
page_header:
  - text: "Confidential information\n\n"
    size: 7
column_header:
  - name: Name
    width: 30
  - name: Age
    width: 10
  - name: Salary
    width: 20
row:
  - value: $F(name)
  - value: $F(age)
  - value: $F(salary)
column_footer:
  - value: "Average:"
  - value: $P{average_age}
  - value: $P{average_salary}
page_footer:
  - text: "Tel: +1 123 456 789"
    size: 7
summary:
  - paragraph:
    - text: "Company address: $P{company_address}"
      size: 10
```
The template consists of the following parts:

**title** - Report title. This section contains the title of the report and is used to clearly indicate the topic or content of the document. This is the first thing the user sees when opening a document.

**page_header** - Page header. This section is used to insert general information that should appear at the top of each report page.

**column_header** - Column headers. This section specifies the names of the columns and their width, which ensures a structured and clear presentation of data in the report tables.

**row** - Data rows. The section is designed to display specific data values. This allows information about each data item to be presented in a consistent and readable manner in its corresponding column.

**column_footer** - Column footer. This section is used to display summary or summary information for column data.

**page_footer** - Page footer. This is where information is placed that should be easily accessible on every page of the report.

**summary** - Report summary. Summary information after the report table.


When iterating over table rows, cells can be accessed through the expression **$F(COLUMN_NAME)**. The report also supports global parameters, which can be accessed through the expression **$P(PARAM_NAME)**.

Example data for the report:
```json
{
   "rows": [
     {
       "name": "John",
       "age": 25,
       "salary": 50000
     },
     {
       "name": "Jane",
       "age": 30,
       "salary": 60000
     },
     {
       "name": "Jim",
       "age": 35,
       "salary": 70000
     }
   ],
   "params": {
     "company_name": "ABCDFG Ltd",
     "company_address": "1234 Elm St, Springfield, IL 62701",
     "average_age": 30,
     "average_salary": 60000
   }
}
```
I did not create a structure and require that the data be placed in it before calling the library. As input I expect just a set of bytes containing JSON in the following format.

To use the Metatron library, add to the Cargo.toml file:
```toml
[dependencies]
metatron = "0.2.1"
```
Example of using the report generator:
```rust
fn main() {
    let template_vec = std::fs::read("report-template.yaml").unwrap();
    let template = std::str::from_utf8(&template_vec).unwrap();
    let data_vec = std::fs::read("report-data.json").unwrap();
    let data = std::str::from_utf8(&data_vec).unwrap();
    let images = HashMap::new();
    let doc = Report::generate(template, data, &images).unwrap();
    let result = shiva::pdf::Transformer::generate(&doc).unwrap();
    std::fs::write("report.pdf",result.0).unwrap();
}
```
Pay attention to the line 'let result = shiva::pdf::Transformer::generate(&doc).unwrap();'. I use the Shiva library to serialize the report,
which currently supports the following formats: Plain text, Markdown, HTML and PDF. In other words,
in the line 'let doc = Report::generate(template, data, &images).unwrap()' we get a report in the Common Document Model (CDM),
supported by the Shiva library. In the future, as the number of document formats supported by this library grows, Metatron will also be able to automatically generate reports in new formats.

The result is [report.pdf](https://github.com/igumnoff/metatron/raw/HEAD/data/report.pdf)

![PDF](https://github.com/igumnoff/metatron/raw/HEAD/pdf.png)

Currently, this project is in the MVP (Minimum Viable Product) stage. In the near future I plan to add:

- Image support
- CLI
- REST API server
- Subreports


The name "Metatron" for the library was inspired by several associative characteristics of the angel Metatron:

1. Recording Angel: Metatron is often associated with office and records. It is believed that it records all actions and events in the Universe. This association with office management and record keeping reflects the primary function of the report generator library.
2. Bridge between the heavenly and the earthly: Metatron is also known as a mediator between the Divine and the human, facilitating communication and understanding. In the context of a library, this can symbolize the transformation of data into understandable and accessible reporting forms


The project's source code is published on [GitHub](https://github.com/igumnoff/metatron).
