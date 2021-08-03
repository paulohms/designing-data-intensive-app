# Chapter 2 - Data Models and Query Languages

Data models are perhaps the most important part of developing software, because they have such a profound effect: not only on how the software is written, but also on how we think about the problem that we are solving.

Most applications are built by layering one data model on top of another. For each layer, the key question is: how is it _represented_ in terms of the next-lower layer.

Good examples are:

- Application developers model the real world (people, merchants, organizations, orders and orders items, etc) using objects or data structures that can be manipulated using APIs using tools like high level programming languages.
- Engineers who develop the programming languages we use designed a way to represent the objects and data structures we create in bytes in the computer memory and machine instructions in a low level programming language.
- Hardware engineers figured out a way to represent the bytes in and instructions in terms of electrical currents, electrical voltages, pulses of light, magnetic fields and more.

Complex application have a lot of intermediary levels, such as APIs built upon APIs, but the basic idea is still the same: each layer hides the complexity of the layers below it by providing a clean **data model**.

There are many different kinds of data models, and every data model embodies assumptions about how it is going to be used. Some kinds of usage are easy and some are not supported; some operations are fast and some perform badly; some data transformations fell natural and some are awkward.

## Relational Model Versus Document Model

The best-known data model today is probably that of **SQL**, based on the relational model proposed by Edgar Codd in 1970: data is organized into _relations_ (called _tables_ in SQL), where each relation is an unordered collection of _tuples_ (_rows_ in SQL).

There's no denial that SQL alongside its relational **Relational Database Management Systems** (RDBMSes) had become the tools of choice for most people who need to store and query data with some kind of regular structure. Innovative at the time, SQL made things like _transactional processing_ (sales or bank transactions, airline reservations and stock-keeping in warehouses) and _batch processing_ (customer invoicing, payroll, reporting) easier to achieve and simpler to do.

Other databases at that time forced application developers to think a lot about the internal representation of the data in the database. The goal of te relational model was to hide that implementation detail behind a cleaner interface. Although alternatives did exist by the time, and many other were created, the relational model came to dominate them all, each competitor rising with a lot of hype in its time, but never lasting in the end.

Computers became vastly more powerful and networked, they started being used for increasingly diverse purposes. And remarkably, relational databases turned out to generalize very well, beyond their original scope of business data processing, to a broad variety of use cases. Much of the web today is still powered by relational databases, be it online publishing, discussion, social networking, ecommerce, games software-as-a-service productivity applications, or much more.

## The Birth of NoSQL

The name “NoSQL” is unfortunate, since it doesn’t actually refer to any particular technology—it was originally intended simply as a catchy Twitter hashtag for a meetup on open source, distributed, nonrelational databases in 2009. A number of interesting database systems are now associated with the #NoSQL hashtag, and it has been retroactively reinterpreted as Not Only SQL.

There are several driving forces behind the adoption of NoSQL databases, including:

- A need for greater scalability than relational databases can easily achieve, including very large datasets or very high write throughput
- A widespread preference for free and open source software over commercial database products
- Specialized query operations that are not well supported by the relational model
- Frustration with the restrictiveness of relational schemas, and a desire for a more
  dynamic and expressive data model

Different applications have different requirements, and the best choice of technology
for one use case may well be different from the best choice for another use case. It
therefore seems likely that in the foreseeable future, relational databases will continue to be used alongside a broad variety of nonrelational datastores—an idea that is sometimes called _polyglot persistence_.

## The Object-Relational Mismatch

A common criticism of the SQL data model: if data is stored in relational tables, an awkward translation layer is required between the objects in the application code and the database model of tables, rows, and columns. The disconnect between the models is sometimes called an _impedance mismatch_.

Object-relational mapping (ORM) frameworks like ActiveRecord and Hibernate reduce the amount of boilerplate code required for this translation layer, but they can’t completely hide the differences between the two models.

The following figure illustrates how a résumé (a LinkedIn profile) could be expressed in a relational schema. The profile as a whole can be identified by a unique identifier, user_id. Fields like first_name and last_name appear exactly once per user, so they can be modeled as columns on the users table. However, most people have had more than one job in their career (positions), and people may have varying numbers of periods of education and any number of pieces of contact information. There is a one-to-many relationship from the user to these items, which can be represented in various ways:

- In the traditional SQL model (prior to SQL:1999), the most common normalized representation is to put positions, education, and contact information in separate tables, with a foreign key reference to the users table, as in Figure 2-1.
- Later versions of the SQL standard added support for structured datatypes and XML data; this allowed multi-valued data to be stored within a single row, with support for querying and indexing inside those documents. These features are supported to varying degrees by Oracle, IBM DB2, MS SQL Server, and PostgreSQL [6, 7]. A JSON datatype is also supported by several databases, including IBM DB2, MySQL, and PostgreSQL.
- A third option is to encode jobs, education, and contact info as a JSON or XML   document, store it on a text column in the database, and let the application interpret its structure and content. In this setup, you typically cannot use the database to query for values inside that encoded column.

![](chapter-2-1.png)

For a data structure like a résumé, which is mostly a self-contained document, a JSON representation can be quite appropriate:

```json
{
  "user_id": 251,
  "first_name": "Bill",
  "last_name": "Gates",
  "summary": "Co-chair of the Bill & Melinda Gates... Active blogger.",
  "region_id": "us:91",
  "industry_id": 131,
  "photo_url": "/p/7/000/253/05b/308dd6e.jpg",
  "positions": [
    {
      "job_title": "Co-chair",
      "organization": "Bill & Melinda Gates Foundation"
    },
    { "job_title": "Co-founder, Chairman", "organization": "Microsoft" }
  ],
  "education": [
    { "school_name": "Harvard University", "start": 1973, "end": 1975 },
    { "school_name": "Lakeside School, Seattle", "start": null, "end": null }
  ],
  "contact_info": {
    "blog": "http://thegatesnotes.com",
    "twitter": "http://twitter.com/BillGates"
  }
}
```

Some developers feel that the JSON model reduces the impedance mismatch between the application code and the storage layer.

The JSON representation has better locality than the multi-table schema in the previous figure. If you want to fetch a profile in the relational example, you need to either perform multiple queries (query each table by user_id) or perform a messy multi way join between the users table and its subordinate tables. In the JSON representation, all the relevant information is in one place, and one query is sufficient.

The one-to-many relationships from the user profile to the user’s positions, educational history, and contact information imply a tree structure in the data, and the JSON representation makes this tree structure explicit.

![](chapter-2-2.png)
