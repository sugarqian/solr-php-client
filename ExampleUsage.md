Table of Contents


# Simple HTML Form #
The following script should provide the kick start needed for anyone unsure of where to start with using the Solr PHP client. Its a simple script that outputs an HTML form that submits to itself. When a query is submitted to it, it will create a Solr service class instance and attempt to pass the query to the Solr. It will then display upto 10 result documents in an unordered list.

```
<?php

// make sure browsers see this page as utf-8 encoded HTML
header('Content-Type: text/html; charset=utf-8');

$limit = 10;
$query = isset($_REQUEST['q']) ? $_REQUEST['q'] : false;
$results = false;

if ($query)
{
  // The Apache Solr Client library should be on the include path
  // which is usually most easily accomplished by placing in the
  // same directory as this script ( . or current directory is a default
  // php include path entry in the php.ini)
  require_once('Apache/Solr/Service.php');

  // create a new solr service instance - host, port, and webapp
  // path (all defaults in this example)
  $solr = new Apache_Solr_Service('localhost', 8180, '/solr/');

  // if magic quotes is enabled then stripslashes will be needed
  if (get_magic_quotes_gpc() == 1)
  {
    $query = stripslashes($query);
  }

  // in production code you'll always want to use a try /catch for any
  // possible exceptions emitted  by searching (i.e. connection
  // problems or a query parsing error)
  try
  {
    $results = $solr->search($query, 0, $limit);
  }
  catch (Exception $e)
  {
    // in production you'd probably log or email this error to an admin
	// and then show a special message to the user but for this example
	// we're going to show the full exception
	die("<html><head><title>SEARCH EXCEPTION</title><body><pre>{$e->__toString()}</pre></body></html>");
  }
}

?>
<html>
  <head>
    <title>PHP Solr Client Example</title>
  </head>
  <body>
    <form  accept-charset="utf-8" method="get">
      <label for="q">Search:</label>
      <input id="q" name="q" type="text" value="<?php echo htmlspecialchars($query, ENT_QUOTES, 'utf-8'); ?>"/>
      <input type="submit"/>
    </form>
<?php

// display results
if ($results)
{
  $total = (int) $results->response->numFound;
  $start = min(1, $total);
  $end = min($limit, $total);
?>
    <div>Results <?php echo $start; ?> - <?php echo $end;?> of <?php echo $total; ?>:</div>
    <ol>
<?php
  // iterate result documents
  foreach ($results->response->docs as $doc)
  {
?>
      <li>
        <table style="border: 1px solid black; text-align: left">
<?php
    // iterate document fields / values
    foreach ($doc as $field => $value)
    {
?>
          <tr>
            <th><?php echo htmlspecialchars($field, ENT_NOQUOTES, 'utf-8'); ?></th>
            <td><?php echo htmlspecialchars($value, ENT_NOQUOTES, 'utf-8'); ?></td>
          </tr>
<?php
    }
?>
        </table>
      </li>
<?php
  }
?>
    </ol>
<?php
}
?>
  </body>
</html>
```

# Using an Alternative HTTP Transport #
Since [r49](https://code.google.com/p/solr-php-client/source/detail?r=49), it is now possible to use an HTTP transport implementation of your choice. An HTTP transport is responsible for making the HTTP based requests to Solr and providing the raw response back for processing. A common request is to use cURL extension for the HTTP requests rather than the built in HTTP url wrapper support, so the client provides two implementations for this.

```
<?php

// construct your transport of choice
// The Curl implementation creates a single cURL session and reuses it for all requests
// which is the suggested way to use cURL, however there was a bug in PHP across a number
// of versions where each set of the URL caused a small memory leak until the session was released
$transportInstance = new Apache_Solr_HttpTransport_Curl();

// CurlNoReuse implementation instead creates and releases a cURL session for each request
$transportInstance = new Apache_Solr_HttpTransport_CurlNoReuse();

// A transport instance can be provided at service construction
$solr = new Apache_Solr_Service('solr.example.com', 8180, '/solr', $transportInstance);

// Or, it can be set at any time. This will be used for subsequent requests.
$solr->setHttpTransport($transportInstance);

// ... use the solr service client as you normally would ..

?>
```

It is also possible to implement the Apache\_Solr\_HttpTransport\_Interface with your own implementation and use that. Use cases would be using your framework of choice's builtin HTTP request methods.

# IBM Developer Works Article #
IBM Developer Works had an article featuring the use of the PHP Solr client shortly after its initial release. This article not only has example PHP code but also briefly covers some Solr setup as well.

> http://www.ibm.com/developerworks/opensource/library/os-php-apachesolr/

NOTE: The syntax error mentioned is no longer present, and was the embarrassing result of me releasing an untested cosmetic change I made. I learned my lesson :)