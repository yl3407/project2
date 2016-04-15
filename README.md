# Project 2

* Assigned: 4/5
* Due: 4/22 8:40AM EST
* (worth 5% of course grade)


In this project, you will put into practice some of the advanced SQL concepts we learned, as well as play with a new datatype.  To do so, you will extend your database from Project 1 to use three interesting features of PostgreSQL -- JSON data types, UDFs, and Triggers.

You will work with your project 1 teammate on this project.  If you have lost your teammate, you are welcome to find a new teammate or do this project alone.  If your teammate is still in the course, you will need to work together!


# JSON

[JSON](https://en.m.wikipedia.org/wiki/Json) is the intergalactic data format language of the web.  In short, it is a textual and human-readable way to encode objects, arrays, numbers, and strings.  Pretty much every programming language supports reading and writing JSON strings. For example, in Python:


    import json
    arr = [ dict(a=i, b=i*100, c="hello") for i in range(5) ]
    jsonencoded = json.dumps(arr)
    print jsonencoded

    # it prints:
    # [{"a": 0, "c": "hello", "b": 0}, {"a": 1, "c": "hello", "b": 100}, {"a": 2, "c": "hello", "b": 200}, {"a": 3, "c": "hello", "b": 300}, {"a": 4, "c": "hello", "b": 400}]

    # you can also load a json string into an object!
    o = json.loads('{"a":0, "c":"yo!"}')
    print o["a"]



The above pretty much describes what JSON is -- objects can be serialized into JSON strings, and JSON strings can be deserialized into objects.  We recommend you learn about it from [wikipedia](https://en.m.wikipedia.org/wiki/Json#Data_types.2C_syntax_and_example) and play around with it in [Python](https://docs.python.org/2/library/json.html) or [Javascript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON).

There are many benefits to storing data in such an unstructured, non-relational fashion.  For one, if you are downloading or scraping data from another source (say the [Twitter API](https://dev.twitter.com/rest/reference/get/search/tweets)), then the result is usually in JSON format.  

Often times, the schema of the data varies, and it is hard to define a single relational schema for the data.  As a somewhat trivial example (that may not align with reality), Tweets from mobile smart phones may include a location attribute (e.g., `location: { lat: -97, lon: 102 }`) describing where the tweet came from, whereas tweets from the web browser or [generated automatically](http://www.botanicalls.com/kits/) may not contain such an attribute.

In these cases, you might not want to take the time to figure out what's missing, what's not missing, and a sensible, multi-table schema to store the data.  Instead, you may want to throw it into a datastore and slowly query and analyze it later.   One option, since JSON data is already encoded as a string, is to  add a `text` column in the database and insert the data.  However, then it's just text and you would need to write a (e.g., Python) application to read the data, transform it into an object, and analyze it -- which kind of defeats the purpose of SQL and the database executor!

Thankfully, modern databases support JSON encoded data, and have an explicit JSON type that supports common JSON operations.  For example, in PostgreSQL:

    -- get the item at index value 2 (e.g., the value 3)
    -- note that the return value is a JSON type
    -- the ::json casts the string '[1,2,3]' into a JSON object
    SELECT '[1,2,3]'::json->2;

    -- so this fails  :(
    SELECT ('[1,2,3]'::json->2)::int;

    -- there is a workaround by using "->>" instead of "->"
    -- "->>" retreives the element as a text value, which _can_ be cast to int
    SELECT ('[1,2,3]'::json->>2)::int;

    -- be careful, the following will throw an error because "a" cannot be cast to an int
    SELECT ('["a", 2, 3]'::json->>0)::int;

    -- get the value of key 'b' in an object
    SELECT '{"a":1,"b":2}'::json->'b'

    -- get the value of the 0th element of the key 'a'
    SELECT ('{"a":[1,2,3,4],"b":5}'::json->'a')->0

    -- create a table with a JSON attribute
    CREATE TABLE test(
      aid int,
      object json
    );

    -- insert a json string into the table!
    INSERT INTO test VALUES(1, '[1,2,3]');
    INSERT INTO test VALUES(2, '{"a":1}');
    INSERT INTO test VALUES(3, '[1,3,2]');

    -- query the table for all rows where the object attribute contains 1 in its first element
    -- the previous approach, accessing the array element 0 fails because of the object:
    -- SELECT object->0 FROM test;
    -- returns: ERROR:  cannot extract array element from a non-array

    -- the workaround is to use a "path" expression, which returns null:
    -- in this case, the query doesn't throw an error for aid = 2 
    -- because '{"a":1}'::json#>>'{0}' returns null, which can be cast to int. (try it out)
    SELECT aid FROM test WHERE (object#>>'{0}')::int = 1;


Take a close look at the [PostgreSQL JSON Documentation](http://www.postgresql.org/docs/9.3/static/functions-json.html) to understand the available functions and operators.


### Your Task

Add a feature  (you don't need to augment your web application from Part 3A) to 
your project 1 database that is sensible given your project application, and uses JSON data.  
You will update the project 1 database directly.

1. Augment your database with one or more JSON type attributes -- you can [alter existing tables](http://www.postgresql.org/docs/9.3/static/sql-altertable.html) or create new ones.
2. Insert at least 5 records into those attributes.
3. Write 2 compelling queries that use data in the JSON attributes.  One should involve a JOIN. (If you do not have a partner, write 1 query).


# UDFs and Triggers

We learned about UDFs and Triggers in lecture, and how they can be used together to perform (potentially arbitrary actions in response to 
modifications in the database.  In this part, implement a trigger that calls a UDF implemented in PL/SQL that would be used in your application.  
The UDF should involve data in your JSON data from the JSON part of this project, and 
it should not be trivial (e.g., the UDF returns its argument, or performs simple arithmetic).
A popular use of Triggers is for complex input validation, or to keep things consistent (automatically computing and caching/copying data).

The [PostgreSQL plpgsql documentation](http://www.postgresql.org/docs/9.3/static/plpgsql.html) may be useful for implementing the UDFs;

### Your Task

You will update the project 1 database:

1. Implement a UDF using plpgsql that involves JSON data from above
2. Implement a Trigger that uses the UDF
3. Write a query that illustrates the trigger/UDF's usage


# Submission

Perform the above tasks and submit using [the Google form](http://goo.gl/forms/ruVySVJjLt).
