
**MapReduce** is a programming model for processing large amounts of data in bulk across many machines, popularized by Google. A limited form of MapReduce is supported by some NoSQL datastores, including **MongoDB** and **CouchDB**, as a mechanism for performing read-only queries across many documents.

MapReduce is neither a declarative query language nor a fully imperative query API (see [[Query Languages for Data]]), but somewhere in between: the logic of the query is expressed with snippets of code, which are called repeatedly by the processing framework. It is based on the *map* (also known as *collect*) and *reduce* (also known as *fold* or *inject*) functions that exist in many functional programming languages.

To give an example, imagine you are a marine biologist, and you add an observation record to your database every time you see animals in the ocean. Now you want to generate a report saying how many sharks you have sighted per month.

```sql
SELECT date_trunc('month', observation_timestamp) AS observation_month,  sum(num_animals) AS total_animals FROM observations WHERE family = 'Sharks' GROUP BY observation_month;
```

The same can be expressed with MongoDB’s MapReduce feature as follows:

```js
db.observations.mapReduce(  function map() {  var year = this.observationTimestamp.getFullYear();
 var month = this.observationTimestamp.getMonth() + 1;
 emit(year + "-" + month, this.numAnimals); 
 },  function reduce(key, values) {  return Array.sum(values); 
 },  {  query: { family: "Sharks" },  out: "monthlySharkReport"  } );
```

The map and reduce functions are somewhat restricted in what they are allowed to do. They must be pure functions, which means they only use the data that is passed to them as input, they cannot perform additional database queries, and they must not have any side effects. These restrictions allow the database to run the functions any‐ where, in any order, and rerun them on failure. However, they are nevertheless powerful: they can parse strings, call library functions, perform calculations, and more.

MapReduce is a fairly low-level programming model for distributed execution on a cluster of machines. Higher-level query languages like SQL can be implemented as a pipeline of MapReduce operations, but there are also many distributed implementations of SQL that don’t use MapReduce. Note there is nothing in SQL that constrains it to running on a single machine, and MapReduce doesn’t have a monopoly on distributed query execution.
Being able to use JavaScript code in the middle of a query is a great feature for advanced queries, but it’s not limited to MapReduce - some SQL databases can be extended with JavaScript functions too.

A usability problem with MapReduce is that you have to write two carefully coordinated JavaScript functions, which is often harder than writing a single query. More‐ over, a declarative query language offers more opportunities for a query optimizer to improve the performance of a query. For these reasons, MongoDB 2.2 added support for a declarative query language called the aggregation pipeline. In this language, the same shark-counting query looks like this:

```json
db.observations.aggregate([  { $match: { family: "Sharks" } },  { $group: {  _id: {  year: { $year: "$observationTimestamp" },  month: { $month: "$observationTimestamp" }  },  totalAnimals: { $sum: "$numAnimals" }  } } ]);
```

The aggregation pipeline language is similar in expressiveness to a subset of SQL, but it uses a JSON-based syntax rather than SQL’s English-sentence-style syntax; the difference is perhaps a matter of taste. The moral of the story is that a NoSQL system may find itself accidentally reinventing SQL, albeit in disguise.