#PHP API Builder
Easily transform MySQL tables into web accessible REST APIs with this mini library for PHP.

[Getting Started](#getting-started) | [Customizing your API](#customizing-your-api) | [Making Requests](#making-requests) | [Using the Data](#using-the-data) | [API Parameter Reference](#api-parameter-reference)


##Getting Started
This PHP API Builder is used to build simple JSON [REST APIs](http://en.wikipedia.org/wiki/Representational_state_transfer) from MySQL databases. With it you (or anyone if you choose to make the API pubic) can access and update data on the web through an easy-to-setup `api.php` page. Using the API parameters provided in this mini lib users can query a database through that `api.php` page using http `GET` parameters and return the results as an array of `JSON` results. A full list of available API parameters is located [below](#api-parameter-reference). 

###How it works

Setting up the API is easy! To add an API to an existing MySQL table simply place this repository's `api_builder_include/` folder and `api_template.php` file in the directory where you want your api page to be located (If you want your api accessible at yourdomain.com/api.php you should put these files in your root directory). Next update the `api_template.php` file to reflect your database info and [API customization](#customizing-your-api). Then save the updated file as `api.php` or whatever you want your API page to be called.

Thats it! You can now access your MySQL database using the [API Builder URL Parameters](#api-parameter-reference) Below is an basic example of how the `$api` object can be setup. For more info and options check out the [API Builder documentation](COME BACK).

###Download

You can direct download a .zip of API Builder by clicking [here](https://github.com/brannondorsey/apibuilder/archive/master.zip).

###Example

Throughout this reference an example database will be used. 
This example table named `users` holds information about imaginary users that belong to an organization. The `api.php` for this example is as follows:

```php
<?php

	 //include the API Builder mini lib
	 require_once("includes/class.API.inc.php");

	 //set page to output JSON
	 header("Content-Type: text/javascript; charset=utf-8");
	 
	  //If API parameters were included in the http request via $_GET...
	  if(isset($_GET) && !empty($_GET)){

	  	//specify the columns that will be output by the api
	  	$columns = "id, 
	  				first_name,
	  				last_name,
	  				email,
	  				phone_number,
	  				city,
	  				state,
	  				bio";

	  	//setup the API
	  	//the API constructor takes parameters in this order: host, database, table, username, password
	  	$api = new $API("localhost", "organization", "users", "username", "secret_password");
		$api->setup($columns);
		$api->set_default_order("last_name");
		$api->set_searchable("first_name, last_name, email, city, state, bio");
		$api->set_default_search_order("last_name");
		$api->set_pretty_print(true);

	  	//sanitize the contents of $_GET to insure that 
	  	//malicious strings cannot disrupt your database
	 	$get_array = Database::clean($_GET);

	 	//output the results of the http request
	 	echo $api->get_JSON_from_GET($get_array);
	}
?>
```

Use the `api_template.php` to create your own api.

##Customizing your API

The API Builder mini lib features many more complex API setups than the one demonstrated in the `api_template.php` file. Some of these features include:

- Making an API Private so that only you can access the data it provides
- Using API keys to track and limit hits-per-day usage to specific users
- And setting API defaults for number of results returned per request, default order to return results, etc…

All `$API` setup methods (excluding the constructor) begin with the word `set`. A full list of these setup methods and a brief description can be viewed below. For more information about each method view the [`class.API.inc.php`](includes/class.API.inc.php) source.

###API Class Setup Methods

Names in __bold__ denote methods that are required to use when building an API. All other methods are optional.

- **`API::__construct($host, $db, $table, $username, $password)`** Instantiates the API object and creates MySQLi database connection.
- **`API::setup($columns)`** tells the API object which column values to use when outputting results objects. The `$columns` parameter is a comma-delimited list of column names that correspond to the column names in your database.
- **`API::set_default_order($column)`** sets the default column for the api to order results by if no 'order_by' parameter is specified in the request.
- `API::set_default_flow($flow)` sets the default [flow](COME BACK) if none is specified in the request.
- `API::set_defualt_output_number($default_output)` sets the number of JSON result objects each API request will output if no 'limit' parameter is included in the request.
- `API::set_max_output_number(int $max_output)` Sets the max number of JSON result objects allowed per request.
- `API::set_pretty_print($boolean)` sets the default JSON output as human readable formatted
- `API::set_searchable($columns)` Enables the [API 'search' parameter](COME BACK) and specifies which columns can be searched. Again the `$columns` parameter is a comma-delimited list of column names that correspond to the column names in your database.
- **`API::set_default_search_order($column)`** sets the default columns for the API to order API 'search' parameter results by if the MySQL FULLTEXT Match...Against… statement is executed in boolean mode (required __only__ if `API::set_searchable()` has enabled columns to be searched).
- `API::set_key_required($boolean)` makes your API require a unique key for each request. For more information on limiting and tracking API users visit the [Protecting your API](#protecting-your-api) section of this documentation.
- `API::set_hit_limit($number_hits_per_day)` sets the number of API hits per API key per day.
- `API::set_private($private_key)` makes the API private (i.e. only you can use it). For more information on this method visit the [Protecting your API](#protecting-your-api) section of this documentation.
- `API::set_no_results_message($message)` sets the error message when no results are found in a request

###Protecting your API

There are two ways of limiting access to your API using the setup methods. The first, and most private, is by setting up your API so that only you (and people who you give your private key to) can access it. The second is to track and limit API usage using API keys that are distributed to your users.

####Making your API Private

To make your API private include the following in when setting up your `api.php` page where `$private_key` is a unique 40 character SHA1: 

```php
$private_key = "4e13b0c28e17087366ac4d67801ae0835bf9e9a1";
$api->set_private($private_key);
```

Then when you make your http request to your API just prepend your key to the request using the [API Private Key Parameter](#private-key-parameter):

`http://fakeorganization.com/api.php?last_name=Renolds&private_key=4e13b0c28e17087366ac4d67801ae0835bf9e9a1`

Viola… your own private API.

####Limiting and Tracking Usage with API Keys

Often API owners supply users with unique API keys to track and limit requests so as not to bog down servers. This is a common practice with the Google, Twitter, and Facebook APIs. The API Builder allows API owners to easily do the same!

#####Database Setup

Because this process requires each user to have their own unique API key to access the API, a new table needs to be made to store information about the users that will be making the requests. This table should be named "users" and should include at least the following columns: "id", "API_key", "API_hits", and "API_hit_date". These table and column names must be exact unless specified otherwise using `API::set_key_required()` optional parameters (see [API setup](#api-setup) below). This SQL database structure can be imported from the [`users_table.sql`](users_table.sql) file.

Alternatively, it is often the case that APIs actually supply data regarding users in the first place. For instance, the Twitter and Facebook APIs deliver data about users! If your API is delivering data from a users table already, and you would like to grant __only__ those users access to your API, you may simply add the "id", "API_key", "API_hits", and "API_hit_date" columns to your existing users table instead of creating a new one.

#####API Setup

Once the new table has been created in your database (or your original table was saving users it was amended to include the required columns) you are ready to setup you API to track and limit user's hits. If you used the default table name "users" and the column names "id", "API_key", "API_hits", and "API_hit_date" then the API setup is easy:

```php
$api->set_key_required(true);
```

If you chose to change the names of any table or andy of the columns then these changes must be specified using `API::set_key_required()`'s optional parameters:

`API::set_key_required($boolean, $users_table_name=false, $key_column_name=false, $hit_count_column_name=false, $hit_date_column_name=false);`

If for instance, you chose not to add a new "users" table to your database because the table used for the API, "fakeorganization_users", is already storing users who you want to give access to the API (like in the case of Twitter or Facebook) the setup would look like this.

```php
$api->set_key_required(true, "fakeorganizations_users");
```

If you also changed the "API_hits" column name to something like "number_of_hits_today" then your API key setup would be:

```php
$api->set_key_required(true, "fakeorganizations_users", false, "number_of_hits_today");
```

__Note:__ Don't forget to specify the unchanged table or column names as `false` to maintain parameter order!

For simplicity is recommended to download and import the "users" table structure without modifying the column names.

All that is required now is for you to fill the "users" table's "API_key"s with 40 character SHA1 and distribute them to your real life users so that they can use your API.

#####Usage and Errors

Once your API has been set to require a key (`API::set_key_required()`) only users who make http requests that contain a valid [API Key Parameter](#api-key-parameter) will be given API results. 

```php
http://fakeorganization.com/api.php?last_name=Renolds&key=1278cf264faca856baf2268e52e2761a75972ec7
```

In the above example the "users" table must include a user row where the value of the "API_key" (or your API's equivalent column) is "1278cf264faca856baf2268e52e2761a75972ec7". In the event that this is not the case, an invalid key is provided, or no key is provided, the following `error` property is returned in the response JSON instead of a `data` array:

```json
{
    "error": "API key is invalid or was not provided"
}
```

In the event that a user has maxed out their API hits for the day (set using `API::set_hit_limit()`) the API would return the following:

```json
{
    "error": "API hit limit reached"
}
```

##Making Requests

Using your API is easy once you learn how it works. 

####Formatting a request

The API Builder queries [MySQL](https://en.wikipedia.org/wiki/MySQL) Databases and so the http requests used to return data is very similar to forming a MySQL `SELECT` query statement. If you have used MySQL before, think of using the api URL parameters as little pieces of a query. For instance, the `limit`, `order_by`, and `flow` (my nickname for MySQL `ORDER BY`'s `DESC` or `ASC`) parameters translate directly into a MySQL statement on your server.

####<a id="example-request"></a>Example Request
     http://fakeorganization.com/api.php?search=Thompson&order_by=id&limit=50
     
The above request would return the 50 newest users who have Thompson included somewhere in their first_name, last_name, email, city, state or bio columns.

####Notable Parameters

- `search` uses a MySQL FULLTEXT search to find the most relevant results in the database to the parameter's value.
- `order_by` returns results ordered by the column name given as the parameter's value.
- `limit` specifies the number of returned results. If not included as a parameter the default value is `25`. Max value is `250`.
- `page` uses a MySQL `OFFSET` to return the contents of a hypothetical "page" of results in the database. Used most effectively when paired with `limit`.

A full list of all API Builder's parameters are specified in the [Parameter Reference](#api-parameter-reference) section of this Documentation.


###Returned JSON

All data returned by your API is wrapped in a `json` object with a `data` array. If there is an error, or no results are found, an `error` variable with a corresponding error message will be returned __instead__ of a `data` property. If your API is setup incorrectly in you `api.php` page a `config_error` array is returned. 

Inside the `data` property is an array of objects that are returned as a result of the URL parameters that will be outlined shortly.

```json
{
    "data": [
        {
            "id": "1035",
            "first_name": "Thomas",
            "last_name": "Robinson",
            "email": "thomasrobinson@gmail.com",
            "phone_number": "8042123478",
            "city": "Richmond",
            "state": "VA",
            "bio": "I am a teacher in the Richmond City Public School System"
        },
        {
            "id": "850",
            "first_name": "George",
            "last_name": "Gregory",
            "email": "gregg@gmail.com",
            "phone_number": "8043703986",
            "city": "Richmond",
            "state": "VA",
            "bio": "I am creative coder from Richmond"
        }
    ]
}
```

__Note:__ The `data` property is always an array of objects even if there is only one result.

##Using the Data

Because the API outputs data using `JSON` the results of an API http request can be loaded into a project written in almost any language. I have chosen to provide brief code examples using `PHP`, however, these code snippets outline the basics of loading and using your your data and easily apply to another language. 


```php
<?php
$city = "Richmond";
$bio = "artist";

$http_request = "http://fakeorganization.com/api.php?city=$city&bio=$bio&limit=10";
	
$json_string = file_get_contents($http_request);
$jsonObj = json_decode($json_string);
	
//loop through each user object inside of the "data" array
foreach($jsonObj->data as $user){
   //do something with each result inside of here...
   //for example, print some of their info to the browser
   echo "This user's first name is " . $user->first_name . "<br/>";
   echo "This user's last name is " . $user->last_name . "<br/>";
   echo "This user's email is " . $user->email;
   echo "<br/>";
}
?>
```

###Error Handling

Often requests to the api return no results because no results were found that met the request's criteria. For this reason it is important to know how to handle the the api `error`. The `JSON` that is returned in this instance is `{"error": "no results found"}`.

Handling `errors` is simple. All that you need to do is check if the `error` property exists in the resulting `JSON` object. If it does execute the code for when an error is present. Otherwise, continue with the program because the request returned at least one result object.

```php
<?php
$city = "Richmond";
$bio = "artist";

$http_request = "http://fakeorganization.com/api.php?city=$city&bio=$bio&limit=10";
	
$json_string = file_get_contents($http_request);
$jsonObj = json_decode($json_string);

//check for an error
if(isset($jsonObj->error)){
	//code for handling the error goes in here...
	//for example, print the error message to the browser
	echo $jsonObj->error;
	}else{
		//execute the code for when user objects are returned…
		//loop through each user object inside of the "data" array
		foreach($jsonObj->data as $user){
		   //do something with each result inside of here...
		   //for example, print some of their info to the browser
		   echo "This user's first name is " . $user->first_name . "<br/>";
		   echo "This user's last name is " . $user->last_name . "<br/>";
		   echo "This user's email is " . $user->email;
		   echo "<br/>";
		}
	}
?>
```

##API Parameter Reference

This section documents in detail all of the API Builder URL parameters currently available to use when making an http request to your API.

It uses a made up table with a setup that is specifed [earlier in this documentation](#example).

###API Key Parameter

If the API has been setup to require a unique key it will need to be included with each request. If the API was set up in this fashion you will not be able to access the API's data unless you have been provided with a valid key. API Builder APIs have this function disabled by default but can be enabled by the API owner with the `API::set_key_required()` method.

Parameter __key:__ `key`

Parameter __value:__ unique 40 character `sha1` key

__Example__: 

     http://fakeorganization.com/api.php?last_name=Renolds&key=8a98253d8b01d4cf8c3fe183ef0862fa69a67b2e
     
__Note:__ Failing to include a valid API key or making more than the allowed requests in a day will throw an `error` object in place of a `data` object.

###Private Key Parameter

Similar to the API Key Parameter the Private Key Parameter limits access to the API. Unlike the API Key however there is only one Private Key allowed per API. This parameter is used if the API is intended to be private (i.e. for only one user, not just limited access as with the API Key Parameter). This function is disabled by default but can be enabled by the API owner with `API::set_private()`.

Parameter __key:__ `key`

Parameter __value:__ unique 40 character `sha1` key

__Example__: 

    http://fakeorganization.com/api.php?last_name=Renolds&private_key=ac3c1017b45b299dbf99ce8470c56b063e24f935


###Column Parameters
Column parameters allow you to query data for a specific value where the parameter key is specified to be the column in your database. Column parameters can be stacked for more specific queries.

Parameter __keys:__ Column name (i.e. `first_name`) to perform query on.

Parameter __values:__ 
Desired lookup `string`, `int`, or `float` that corresponds to the column name in the database as specified by the parameter's key.

__Example:__

      http://fakeorganization.com/api.php?last_name=thompson&limit=10
      
This example request would return up to 10 users who's last name are thompson ordered alphabetically by last name.

__Notes:__ The column parameter's are overridden if a `search` parameter is specified. 

###Search Parameter
The `search` parameter uses a  MySQL `FULLTEXT` [Match()… Against()…](http://dev.mysql.com/doc/refman/5.5/en/fulltext-search.html#function_match) search to find the most relevant results to the searched string. 

Parameter __key:__ `search`

Parameter __value:__ desired query `string`

__Example:__

	http://fakeorganization.com/api.php?search=design

__Notes:__ `search` results are automatically ordered by relevancy, or if relevancy is found to be arbitrary, by `timestamp`. The `order_by` parameter cannot be used when the `search` parameter is specified. More on why below…

Default Match()…Against()… MySQL statements search databases using a 50% similarity threshold. This means that if a searched string appears in more than half of the rows in the database the search will ignore it. Because it is possible that webpages will have similar tags, I have built the api to automatically re-search `IN BOOLEAN MODE` if no results are found in the first try. If results are found in the second search they are ordered by `timestamp`.

###Order By Parameter

This parameter is used with the column parameters to sort the returned users by the specified value. If `order_by` is not specified its value defaults to the value set by the API's `API::set_default_order()` in the `api.php` page. Order by will be ignored when the `search` parameter is specified.

Parameter __key:__ `order_by`

Parameter __value:__ Column name (i.e. `id`) to order by

__Example:__

	http://fakeorganization.com/api.php?state=VA&order_by=id&limit=15

This request returns the 15 most recent users from Virginia.

###Flow Parameter

This parameter specifies the MySQL `ASC` and `DESC` options that follow the `ORDER BY` statement. If `flow` is not specified it defaults to `DESC`.

Parameter __key:__ `flow`

Parameter __value:__ `ASC` or `DESC`

__Example:__

	http://fakeorganization.com/api.php?state=VA&order_by=id&limit=15&flow=asc
	
This request returns the 15 __least recent__ users from Virginia.		
__Notes:__ `flow`'s values are case insensitive.

###Limit Parameter

The `limit` parameter works similarly to MySQL `LIMIT`. It specifies the max number of users to be returned. The default value, if unspecified is `25`. The default max value of results that can be returned in one request is `250`. 

Parameter __key:__ `limit`

Parameter __value:__ `int` between `1-250` or between `1` and the max value specified by the `api.php` page's `API::set_max_output_number()`.

__Example:__

	http://fakeorganization.com/api.php?state=IL&limit=5

Returns the 5 most recent users from Illinois.

###Page Parameter

The page parameter is used for results pagination. It keeps track of what set (or page) of results are returned. This is similar to the [MySQL OFFSET statement](http://dev.mysql.com/doc/refman/5.0/en/select.html). If not specified the page value will default to `1`.

Parameter __key:__ `page`

Parameter __value:__ `int` greater than `0`

__EXAMPLE:__ 

	http://fakeorganization.com/api.php?search=programmer&limit=7&page=3&order_by=id&flow=asc	
This request will return the 3rd "page" of `search` results. 

For instance, in the unlikely example that all users had "programming" in their bios, setting `page=1` would return webpages with id's `1-7`, setting `page=2` would yield `8-14`, etc…

__Note:__ The MySQL `OFFSET` is calculated server side by multiplying the value of `limit` by the value of `page` minus one. 

###Exact Parameter

The exact parameter is used in conjunction with the column parameters and specifies whether or not their values are queried with relative or exact accuracy. If not included in the URL request the `exact` parameter defaults to `false`.

Parameter __key:__ `exact`

Parameter __values:__ `TRUE	` and `FALSE`

__Example:__

	http://fakeorganization.com/api.php?id=10&exact=TRUE

This request will limit the returned results to the users whose id is __exactly__ 10. If the `exact` parameter was not specified, or was set to `FALSE`, the same request could also return users whose id's have 10 in them (i.e. 1310, 10488, 100 etc…). including `exact=true` parameter in an api http request is equivalent to a `MySQL` `LIKE` statement.
	
__Notes:__ `exact`'s values are case insensitive.

###Exclude Parameter

The exclude parameter is used in conjunction with the column parameters to exclude one or more specific webpage's from a query.

Parameter __key:__ `exclude`

Parameter __values:__ a comma-delimited list of excluded users `id`'s

__Example:__

	http://fakeorganization.com/api.php?email=@gmail.com&exclude=5,137,1489&limit=50

This example will return 50 users other than numbers `5`, `137`, and `1489` who use gmail ordered by last name. 

###Count Only Parameter

The `count only` parameter differs from all of the other API Builder parameters as it __does not__ return an array of result objects. Instead, it returns a single object as the first element in the `data` array. This object has only one property, `count`, where the corresponding `string` value describes the number of results returned by the rest of the url parameters. If the `count_only` parameter is not specified the default value is `FALSE`. When `count_only` is set to `TRUE` the request will __only__ evaluate and return the number of results found by the rest of the url parameters and the request __will not__ return any user data.

Parameter __key:__ `count_only`

Parameter __values:__ `	TRUE` or `FALSE`

__EXAMPLE:__

     //request
     http://fakeorganization.com/api.php?first_name=Thomas&exact=true&count_only=true
     
     //returns
     {
      "data":[
        {
        "count":"45"
        }]
     }
     
This request returns the number of users that have the first name "Thomas". The count value is returned as __a `string`__.

__Note:__ The value of `count_only` is case insensitive.

###Pretty Print Parameter

The Pretty Print Parameter returns a response with indentations and line breaks. If Pretty Print is set to `TRUE` the results returned by the server will be human readable (pretty printed). With most API setups Pretty Print will be enabled by default but the API owner may have this feature disabled. Either way Pretty Print can be turned on or off with this parameter. 

Parameter __key:__ `pretty_print`

Parameter __value:__ `TRUE` or `FALSE`

__Example:__

	http://fakeorganization.com/api.php?email=@gmail.com&pretty_print=false
	
__Note:__ The value of `count_only` is case insensitive. If large amounts of data are being transfered and human readibility is unimportant it is suggested to disable pretty print so as to enable faster API requests and data parsing.

##License and Credit

The API Builder PHP Mini Library is developed and maintained by [Brannon Dorsey](http://brannondorsey.com) and is published under the [MIT License](license.txt). If you notice any bugs, have any questions, or would like to help me with development please submit an issue or pull request, write about it on the wiki, or [contact me](mailto:brannon@brannondorsey.com).