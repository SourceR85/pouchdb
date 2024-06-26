---
index: 5
layout: guide
title: Working with documents
sidebar: guides_nav.html
---

{% include anchor.html title="What's a document?" hash="whats-a–document" %}

PouchDB is a NoSQL database, meaning that you store unstructured *documents* rather than explicitly specifying a schema with rows, tables, and all that jazz.

A document might look like this:

```js
{
  "_id": "mittens",
  "name": "Mittens",
  "occupation": "kitten",
  "age": 3,
  "hobbies": [
    "playing with balls of yarn",
    "chasing laser pointers",
    "lookin' hella cute"
  ]
}
```

If you come from a SQL background, this handy conversion chart may help:

<div class="table-responsive">
<table class="table">
<tr>
  <th>SQL concept</th>
  <th>PouchDB concept</th>
</tr>
<tr>
  <td>table</td>
  <td><em>no equivalent</em></td>
</tr>
<tr>
  <td>row</td>
  <td>document</td>
</tr>

<tr>
  <td>column</td>
  <td>field</td>
</tr>
<tr>
  <td>primary key</td>
  <td>primary key (<code>_id</code>)</td>
</tr>
<tr>
  <td>index</td>
  <td>view</td>
</tr>
</table>
</div>

We'll discuss these concepts later on.

{% include anchor.html title="Storing a document" hash="storing-a–document" %}

To store a document, you simply `put` it:

```js
const doc = {
  "_id": "mittens",
  "name": "Mittens",
  "occupation": "kitten",
  "age": 3,
  "hobbies": [
    "playing with balls of yarn",
    "chasing laser pointers",
    "lookin' hella cute"
  ]
};
db.put(doc);
```

Whenever you `put()` a document, it must have an `_id` field so that you can retrieve it later.

So now let's `get()` the document by using its `_id`:

```js
db.get('mittens').then(function (doc) {
  console.log(doc);
});
```

You should see:

```js
{
  "name": "Mittens",
  "occupation": "kitten",
  "age": 3,
  "hobbies": [
    "playing with balls of yarn",
    "chasing laser pointers",
    "lookin' hella cute"
  ],
  "_id": "mittens",
  "_rev": "1-bea5fa18e06522d12026f4aee6b15ee4"
}
```

You can see a **[live example](http://bl.ocks.org/nolanlawson/c02bba75247012afb1bf)** of this code.

The document looks exactly the same as when we put it, except... aha! What is this? There is a new field, `_rev`, that contains what looks like garbage. PouchDB gots some 'splainin' to do.

{% include anchor.html title="Understanding revisions (`_rev`)" hash="understanding-revisions-rev" %}

The new field, `_rev` is the *revision marker*. It is a randomly-generated ID that changes whenever a document is created or updated.

Unlike most other databases, whenever you update a document in PouchDB or CouchDB, you must present the *entire document* along with its current *revision marker*.

For instance, to increment Mittens' age to 4, we would do:

```js
doc.age = 4;
doc._rev = "1-bea5fa18e06522d12026f4aee6b15ee4";
db.put(doc);
```

If you fail to include the correct `_rev`, you will get the following sad error:

```js
{
  "status": 409,
  "name": "conflict",
  "message": "Document update conflict"
}
```

`HTTP 409` is a standard HTTP error message that indicates a conflict.

{% include anchor.html title="Updating documents correctly" hash="updating-documents–correctly" %}

So to update Mittens' age, we will first need to fetch Mittens from the database, to ensure that we have the correct `_rev` before we put them back. We don't need to manually assign the `_rev` value here (like we did above), as it is already in the `doc` we're fetching.

```js
// fetch mittens
db.get('mittens').then(function (doc) {
  // update their age
  doc.age = 4;
  // put them back
  return db.put(doc);
}).then(function () {
  // fetch mittens again
  return db.get('mittens');
}).then(function (doc) {
  console.log(doc);
});
```

You can see a **[live example](http://bl.ocks.org/nolanlawson/d6daa02ca3875d1222dd)** of this code.

{% include alert/start.html variant="info" %}

Don't worry if the structure of this code seems strange! It's using <strong>promises</strong>, which will be discussed in the next chapter.

{% include alert/end.html %}

Now you should see the following:

```js
{
  "name": "Mittens",
  "occupation": "kitten",
  "age": 4,
  "hobbies": [
    "playing with balls of yarn",
    "chasing laser pointers",
    "lookin' hella cute"
  ],
  "_id": "mittens",
  "_rev": "2-3e3fd988b331193beeeea2d4221b57e7"
}
```

As you can see, we have successfully updated Mittens' age to 4 (they grow up so fast!), and their revision marker has also changed to `"2-3e3fd988b331193beeeea2d4221b57e7"`. If we wanted to increment their age to 5, we would need to supply this new revision marker.

{% include anchor.html title="Related API documentation" hash="related-api-documentation" %}

* [get()](/api.html#fetch_document)
* [put()](/api.html#create_document)

{% include anchor.html title="Next" hash="next" %}

Now that you understand a bit about how to create and update documents, let's take a small detour to talk about asynchronous code.
