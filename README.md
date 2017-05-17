Export JS Template
==================

Microservice to export data using custom defined SPARQL queries.

## Using the template
Run the `semtech/mu-export-js-template` service and mount an export configuration in `/config/export.js`.

The `export.js` file defines an array of export configuration objects as follows:

```javascript
export default [
    {
        path: '/example',
        format: 'text/csv',
        query: `SELECT * FROM <http://mu.semte.ch/application> WHERE { ?s ?p ?o } LIMIT 100`,
        file: 'export-example.csv'
    },
    ...
];

```

An export configuration object consists of the following properties:
* [REQUIRED] `path`: endpoint on which the export will be exposed
* [REQUIRED] `format`: format of the export (`application/json`, `text/csv`, `text/turtle`, etc.)
* [REQUIRED] `query`: the SPARQL query to execute. This may be a `SELECT` or a `CONSTRUCT` query.
* [OPTIONAL] `file`: name of the downloaded file. The filename may also be provided through a query param on the export request (e.g. `/example?file=my-name.csv`). If no filename is provided, the export will be displayed inline in the browser.

## Example docker-compose

```yaml
version: '2',
services:
  export:
    image: semtech/mu-export-js-template:0.2.0
    volumes:
      - ./:/config
    links:
      - database:database
```

Note: if you extend the `semtech/mu-export-js-template` with an `export.js` file instead of mounting a volume in `/config`, the `ONBUILD` commands of the `semtech/mu-javascript-template` will not be executed in your image since `ONBUILD` is only executed one level deep.

## Using variables in the export query
You can also define variables in your SPARQL export query which will be replaced with query param values at runtime. Therefore, we use [ES6's tagged template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#Tagged_template_literals).

Import the `template` tag function from `/app/template.js` in your `export.js` and apply the tag function on your query. You can then define variables in your SPARQL query using `${'myVariable'}`. At runtime the variable will be replaced with the value provided in the `myVariable` query param of the request.

An example `export.js` file may look as follows:

```javascript
export default [
    {
        path: '/example',
        format: 'text/csv',
        query: `SELECT * FROM <http://mu.semte.ch/application> WHERE { ?s a ${'class'} } LIMIT 100`,
        file: 'export-example.csv'
    },
    ...
];

```

A GET request on `/example?class=<http://xmlns.com/foaf/0.1/Person>` will export the first 100 URIs of type `foaf:Person`.
