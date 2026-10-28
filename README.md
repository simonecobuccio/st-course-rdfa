# Semantic Technologies course page in RDFa and SPARQL

We wrote a small web page that describes the Semantic Technologies course of our master, annotated it with **RDFa** using the **schema.org** vocabulary, turned the annotations into an RDF graph in **Turtle**, and queried that graph with **SPARQL**.

---
## The assignment

This was the group exam of the Semantic Technologies course. The course covered the RDF data model, RDFa, SPARQL, and a little of RDFS and OWL. The exam was a live session: groups of about four people presented their work on a shared screen, and the professor asked questions to the group and to each member.

Before the exam, each group had to:

- choose a text and the type of resource it describes (a recipe, an event, and so on);
- annotate the text in RDFa, using the tools seen in class, so that it fills a Google rich result card;
- convert the annotated page to Turtle syntax;
- prepare some meaningful and complex SPARQL queries on the resulting graph.

During the exam, each group had to:

- annotate the text in RDFa live, explaining each step;
- extract the RDF triples and load them in the RDF Playground;
- run the prepared SPARQL queries and explain them;
- run other SPARQL queries assigned live by the professor;
- answer questions on the technologies of the course (RDF, RDFa, SPARQL, some OWL).

There were also a few rules. The RDFa had to be readable by the schema.org validator, the Turtle graph and the queries had to run in the RDF Playground, and every member had to contribute in equal parts and be able to explain any part of the work.

---
## What we did

### 1. Domain

We chose to describe the Semantic Technologies course itself, using the schema.org types `Course` and `CourseInstance`. The course is a good fit because it has a bit of everything: plain text (name, description, prerequisites), numbers (credits, prices, ratings), dates (start and end of each edition), nested entities (the provider, the instructor, the offers, the rating) and some missing data. In particular, the 2027 edition only has a start date, which was useful later to show how `OPTIONAL` works.

The graph is built around one `Course` with:

- general properties: name, description, course code, language, number of credits and prerequisites;
- a `CollegeOrUniversity` as provider (Politecnico di Milano, Graduate School of Management);
- two `CourseInstance` editions: the 2026 one (13 to 19 May, Milan, onsite, 15 hours, with a `Person` as instructor) and the 2027 one (start date only);
- two `Offer` entities: €0 for students of the master and €800 for external participants;
- an `AggregateRating` of 4.6 out of 5 from 134 reviews.

### 2. RDFa annotation

`course.html` is a normal page written for a human reader, with the annotations added on top of the existing text. We used the main RDFa attributes:

| Attribute  | What it does in the page |
| ---------- | ------------------------ |
| `vocab`    | sets schema.org as the default vocabulary on the `<body>` |
| `typeof`   | declares the type of a new entity (`Course`, `CourseInstance`, `Person`, `Offer`, ...) |
| `resource` | gives an identifier to an entity, so the two editions become `#instance2026` and `#instance2027` |
| `about`    | goes back to an entity already declared, to add more properties to it later in the text |
| `property` | attaches a value to the current subject |
| `content`  | overrides the visible text, e.g. "in-person" in the text becomes `Onsite` in the data |
| `datatype` | types the literals (`xsd:date`, `xsd:decimal`, `xsd:integer`) so that dates and numbers can be compared in SPARQL |

We checked the page with the [schema.org validator](https://validator.schema.org/) and with Google's Rich Results Test, which shows a preview of the rich result card.

### 3. RDF graph

We converted the page to Turtle with the [EasyRdf converter](https://www.easyrdf.org/converter). The result is `course.ttl`, a graph of 40 triples. The course itself is a blank node, because the page gives no identifier to it, while the two editions have their own URIs thanks to the `resource` attribute.

### 4. SPARQL queries

`queries.rq` contains the queries we prepared. The first eight are the ones in the presentation, each one showing a different feature of SPARQL. The last two are extras we prepared for the live questions.

| # | Query | Feature | Result on our graph |
| - | ----- | ------- | ------------------- |
| 1 | Core properties of the course | `SELECT` with several patterns on the same subject | name, 2 credits, code, `en` |
| 2 | Instructor of the course | join across three entities, `FILTER` with `STR()` | Matteo Belcao and his email |
| 3 | Paid offers | numeric `FILTER` on `xsd:decimal` | the €800 offer |
| 4 | Editions after 2026 | `FILTER` on `xsd:date` | the 2027 edition |
| 5 | All editions with their details | separate `OPTIONAL` blocks | both editions, with empty cells for 2027 |
| 6 | Statistics of the offers | `COUNT`, `AVG`, `MIN`, `MAX` with `GROUP BY` | 2 offers, average €400, from €0 to €800 |
| 7 | Code and language in one column | `UNION` | `ST-BADS-GSOM-POLIMI` and `en` |
| 8 | Cheapest offer | `ORDER BY` and `LIMIT` | €0 |
| 9 | Instructors as a new class (extra) | `CONSTRUCT` | two new triples for the instructor |
| 10 | Prerequisites (extra) | multi-valued property | three rows, one per prerequisite |

Queries 4 and 5 use `STR()` in the projection only to print the values without the datatype, which makes the result table easier to read.

The [slideshow](slideshow.pdf) shows the course model, the annotated HTML, the Turtle graph, the rich result preview and every query with its output.

---
## Repository structure

```
st-course-rdfa/
├── course.html      course page annotated in RDFa (schema.org)
├── course.ttl       RDF graph extracted from the page, in Turtle
├── queries.rq       SPARQL queries on the graph
├── slideshow.pdf    exam presentation
├── LICENSE
└── README.md
```

---
## How to run it

Everything runs in the browser, so nothing needs to be installed.

1. **Check the annotations.** Paste the content of `course.html` in the [schema.org validator](https://validator.schema.org/) (option "Code snippet"). It should find one `Course` with its nested entities.
2. **Extract the triples.** Paste the same HTML in the [EasyRdf converter](https://www.easyrdf.org/converter), with RDFa as input and Turtle as output. The result is equivalent to `course.ttl`.
3. **Run the queries.** Open the [RDF Playground](https://rdfplayground.dcc.uchile.cl/), load `course.ttl` as data, and run the queries in `queries.rq` one at a time, with the `PREFIX` lines on top.

If you prefer Python, the graph and the queries also work with [rdflib](https://rdflib.readthedocs.io/).

---
## Authors

Group project (Group 6) for the Semantic Technologies course of the **Master in Business Analytics and Data Science** at **Politecnico di Milano** GSoM 2025/2026.

- Alexandre Benjamin Cambier
- Simone Cobuccio
- Giancarlo Ferrara
- Davide Tissino

---
## License

The code is released under the [MIT License](LICENSE).
