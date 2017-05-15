---
layout: post
title: Apigility RPC for IBM i DB2
excerpt: A simple example of an Apigility RPC calling an IBM i DB2 Stored Procedure.
image: /images/rpc-ibm-db2/stored-procedure-action.png
categories:
    - apigility
    - ibm i
    - db2
---

* TOC
{:toc}

### Apigility Setup

This example assumes a setup similar to the one described in
[Installing and Using Apigility on IBM i](/Installing-and-Using-Apigility-on-IBM-i/). The example repository can be
found on [github](https://github.com/jbh/apigility-ibm-i).

### Create New RPC

Create a new RPC through the Apigility Admin Interface like normal. This will generate a Factory and Controller for the
RPC service. Once that is done, I like to quickly set the RPC HTTP Method to be POST.

![New RPC Service](/images/rpc-ibm-db2/new-service.png)

![Allowed HTTP Methods](/images/rpc-ibm-db2/allowed-http-methods.png)

### Define Adapter in Controller

The controller needs our database adapter in order to call stored procedures. We'll define that with a private variable
and initialize it in the controller's constructor like so:

```php
<?php
namespace ApiNamespace\V1\Rpc\StoredProcedureName;

use Zend\Mvc\Controller\AbstractActionController;
use ZF\ApiProblem\ApiProblem;
use ZF\ApiProblem\ApiProblemResponse;

class StoredProcedureNameController extends AbstractActionController
{
    /**
     * @var \Zend\Db\Adapter\Adapter $db
     */
    private $db;

    public function __construct($db) {
        $this->db = $db;
    }
}
```

### Use Factory to Inject Adapter

The defined database adapter, in this case `ibmdb`, needs to be injected to the controller through the RPC's factory.
We can do that with the service manager:

```php
<?php
namespace ApiNamespace\V1\Rpc\StoredProcedureName;

class StoredProcedureNameControllerFactory
{
    /**
     * @param \Zend\ServiceManager\ServiceManager $services
     * @return StoredProcedureNameController
     */
    public function __invoke($services)
    {
        $db = $services->get('ibmdb');
        return new StoredProcedureNameController($db);
    }
}
```

### Call the Stored Procedure

Now that we have access to the database adapter, we can use the same connection resource in order to call our stored
procedure. This allows us to use `IN`, `OUT`, and `INOUT` DB2 parameters.

```php
<?php
namespace ApiNamespace\V1\Rpc\StoredProcedureName;

use Zend\Mvc\Controller\AbstractActionController;
use ZF\ApiProblem\ApiProblem;
use ZF\ApiProblem\ApiProblemResponse;

class StoredProcedureNameController extends AbstractActionController
{
    /**
     * @var \Zend\Db\Adapter\Adapter $db
     */
    private $db;

    public function __construct($db) {
        $this->db = $db;
    }

    public function storedProcedureNameAction()
    {
        $data = $this->bodyParams();

        $parameter1 = $data['parameter1'];
        $parameter2 = $data['parameter2'];
        $parameter3 = $data['parameter3'];

        $sql = "CALL LIBRARY.STORED_PROCEDURE_NAME(?, ?, ?)";

        /* @var \Zend\Db\Adapter\Driver\IbmDb2\Statement $stmt */
        $stmt = $this->db->createStatement();
        $stmt->prepare($sql);
        $stmtResource = $stmt->getResource();
        db2_bind_param($stmtResource, 1, 'parameter1', DB2_PARAM_INOUT);
        db2_bind_param($stmtResource, 2, 'parameter2', DB2_PARAM_INOUT);
        db2_bind_param($stmtResource, 3, 'parameter3', DB2_PARAM_INOUT);
        db2_execute($stmtResource);

        $output = [
            'parameter1' => $parameter1,
            'parameter2' => $parameter2,
            'parameter3' => $parameter3,
        ];

        return $output;
    }
}
```

There we are! We have a simple and repeatable way to call stored procedures with RPCs.

I made this article because I found it difficult to find an example where the developer was using the
same database resource to call a stored procedure, as I haven't found a way to bind parameters through the ZF2
IBM DB2 Statement object in a way that gives the output to the bound parameter. My solution was to get the statement
resource and use it with the built in PHP DB2 functions.
