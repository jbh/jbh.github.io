---
layout: post
title: Apigility Entity & Mapper Tips
excerpt: |
    A guide for getting started with Apigilty, Entities, Mappers, and everything else needed for a RESTful API.
image: /images/apigility-entity-mapper-tips/ecommerce-user-fields.png
categories:
    - apigility
    - ibm i
    - db2
author:
    name: Josh Hall
    twitter: tweetjbh
---

* TOC
{:toc}

### Create a Simple Schema and Table

To get us started, we'll need a source of data for our service. I'm going to use IBM i DB2 for this example,
since that is primarily what I work on. Below is SQL to define a simple table and populate it with some data.

```sql
CREATE SCHEMA APIGILITY ;

/* `FOR COLUMN` allows a long name and short name to be assigned to the column. */
CREATE TABLE APIGILITY/ECOMMERCE_USERS (
    ID FOR COLUMN EUUSERID BIGINT GENERATED ALWAYS AS IDENTITY (
        START WITH 1
        INCREMENT BY 1,
        NO ORDER,
        NO CYCLE,
        NO MINVALUE,
        NO MAXVALUE,
        CACHE 20
    ),

    USERNAME   FOR COLUMN EUUSER  VARCHAR(64) NOT NULL,
    EMAIL      FOR COLUMN EUEMAIL VARCHAR(64) NOT NULL,
    FIRST_NAME FOR COLUMN EUFNAME VARCHAR(32),
    LAST_NAME  FOR COLUMN EULNAME VARCHAR(32),

    CREATED_AT FOR COLUMN EUCRT TIMESTAMP NOT NULL
        DEFAULT CURRENT TIMESTAMP,
    MODIFIED_AT FOR COLUMN EUMOD TIMESTAMP
        FOR EACH ROW ON UPDATE AS ROW CHANGE TIMESTAMP NOT NULL,    

    PRIMARY KEY (ID),
    UNIQUE (EMAIL)
) RCDFMT EUFMT ;

/* For Greenscren applications */
RENAME TABLE APIGILITY/ECOMMERCE_USERS
  TO SYSTEM NAME ECOMMUSERS ;

LABEL ON TABLE APIGILITY/ECOMMERCE_USERS IS 'ECOMMERCE USERS' ;

LABEL ON COLUMN APIGILITY/ECOMMERCE_USERS (
    EUUSERID TEXT IS 'ID',
    EUUSER   TEXT IS 'USERNAME',
    EUEMAIL  TEXT IS 'EMAIL',
    EUFNAME  TEXT IS 'FIRST NAME',
    EULNAME  TEXT IS 'LAST NAME',
    EUCRT    TEXT IS 'CREATED DATE',
    EUMOD    TEXT IS 'LAST MODIFIED'
) ;

/* Inserts to populate ECOMMERCE_USERS with example data */
INSERT INTO APIGILITY/ECOMMERCE_USERS (USERNAME, EMAIL, FIRST_NAME, LAST_NAME)
VALUES ('pip', 'pip@example.com', 'Pip', 'Jenkins') ;

INSERT INTO APIGILITY/ECOMMERCE_USERS (USERNAME, EMAIL, FIRST_NAME, LAST_NAME)
VALUES ('eleanor', 'eleanor@example.com', 'Eleanor', 'Fant') ;

INSERT INTO APIGILITY/ECOMMERCE_USERS (USERNAME, EMAIL, FIRST_NAME, LAST_NAME)
VALUES ('natalya', 'natalya@example.com', 'Natalya', 'Undergrowth') ;
```

If all ran well, you should end up with a populated table like so:

![Ecommerce Users Table](/images/apigility-entity-mapper-tips/ecommerce-users-table.png)

### Install Apigility

Please follow the instructions outlined in
[Installing and Using Apigility on IBM i](/Installing-and-Using-Apigility-on-IBM-i) up until the "Poor Man's Rest"
portion. We're going to describe a not-so-Poor Man's Rest in this article. We will be using Mappers, Entities,
and Collections to build a more mature, RESTful API.

#### Double Check Apigility Configuration

A few configurations are important to the way this tutorial handles entities with the IBM i DB2 data source. One
important configuration to note is the `'db2_attr_case' => DB2_CASE_LOWER` driver option for the DB2 adapter found in
`local.php`, or wherever the database connection is configured. Make sure this is set to lowercase all columns. This
helps keep the Mapper and Entity consistent.

To elaborate, the Mapper will be dealing with data directly, while the Entity is just an Object representation of the
data. If we want to keep the properties of the entity lowercase, which we do, we will need all the returned column names
to be lowercase as well.

### Create the Rest Service

We have Apigility installed on our system as well as an example table called `ECOMMERCE_USERS` in our `APIGILITY`
schema. It is time to create a new rest service through the Apigility admin interface. Let's call it `EcommerceUser`.

![New Ecommerce Api Service](/images/apigility-entity-mapper-tips/new-ecommerce-api-service.png)

I like to go ahead and define my fields at this point. It just seems like a natural first step to take after
creating the service.

Defining a field is simple. Each field represents a column in your database, and it should have the same constraints.
Constraints can be defined through the validator and filter options. For this example, we're going to simply require
that username and email be required.

![Ecommerce User Fields](/images/apigility-entity-mapper-tips/ecommerce-user-fields.png)

Everything should be "working" fine up until this point. You should be able to call
`GET api.ibmiserver.com/ecommerce-user` through Postman, for example, and you'll at least get something like, "The GET
method has not been defined for collections". This is a good sign. It just means we still have to flesh out our service.

### Create the Entity

Entities are simply an object representation of our data. It should not do anything with the data. It should not have
any logic within it. It simply represents our data as an object. With that in mind, our Entity will need a property
for each field. It will also need one method for converting the object to an array, and one method for populating the
entity.

```php
<?php
// module/Test/src/V1/Rest/EcommerceUser/EcommerceUserEntity.php
namespace Test\V1\Rest\EcommerceUser;

class EcommerceUserEntity
{
    public $id;
    public $username;
    public $email;
    public $first_name;
    public $last_name;
    public $created_at;
    public $modified_at;

    public function getArrayCopy()
    {
        return get_object_vars($this);
    }

    public function populate($data)
    {
        foreach ($data as $key => $value) {
            $this->{$key} = $value;
        }
    }
}
```

> Keep in mind that `populate` is simply there to populate our entity. It is not a rigid function. It can be changed
to populate your data in whatever fashion is necessary. This example's populate function is this simple because
of the consistency we made sure of earlier. Since this entity's properties match our data exactly, it can be
populated this easily. I've seen populate functions get much more convoluted.

### Override DbSelect

Earlier on I mentioned how the DB2 configuration option `'db2_attr_case' => DB2_CASE_LOWER` forced all returned column
names to be lowercase. That helps with consistency between the mapper and the entity. While this helps with that, it
also breaks some of the third-party functionality. Specifically, in this case, I discovered it broke the result count
necessary for the pagination. The count relies on the column it defines (`C`) to stay uppercase. The DB2 configuration
forces it to lowercase. In order to fix this, I simply extended the Zend Framework `DbSelect` class and overrode the
`count()` method.

This class could be added directly to the mapper itself, but for simplicity, good practice, and reusability, I suggest
putting it in a composer-loaded file. Until I'm able to get around to making another private composer package, I usually
put these into a temporary directory and autoload them through the `composer.json`. For example, I might put
`LowercaseDbSelect` into a folder structure like `phplib/Adapter` at the root of the project.

```php
<?php
// phplib/Adapter/LowercaseDbSelect.php
namespace CompanyNamespace\Adapter;

use Zend\Paginator\Adapter\DbSelect;

class LowercaseDbSelect extends DbSelect
{
    public function count()
    {
        if ($this->rowCount !== null) {
            return $this->rowCount;
        }

        $select = $this->getSelectCount();

        $statement = $this->sql->prepareStatementForSqlObject($select);
        $result    = $statement->execute();
        $row       = $result->current();

        $this->rowCount = (int) $row['c'];

        return $this->rowCount;
    }
}
```

This would be autoloaded through the `composer.json` like so:

```json
{
    "autoload": {
        "psr-4": {
            "CompanyNamespace\\": "phplib/"
        }
    },
}
```

### Create the Mapper

We have Apigility, a Rest Service, and an Entity for said Rest Service in place, but how are we supposed to actually
manipulate the data? We now need a mapper to map the data to the entity.

Mappers allow us to connect the entity with data. If the API user is getting data, the mapper will select the data,
attach it to an **entity**, and return the **entity** to the user. If the API user is manipulating data, the mapper will
take a manipulated **entity**, update the data accordingly, and return the result to the user. I emphasize entity here
in hopes to emphasize the importance that entity is the object being passed around. The mapper returns an entity
or collection with fetch, and it will take an entity for inserting and updating data.

Sounds great, right? So let's make one.

```php
<?php
// module/Test/src/V1/Rest/EcommerceUser/EcommerceUserMapper.php
namespace Test\V1\Rest\EcommerceUser;

use Zend\Db\Sql\Select;
use Zend\Db\Sql\Delete;
use Zend\Db\Adapter\Adapter;
use CompanyNamespace\Adapter\LowercaseDbSelect;

class EcommerceUserMapper
{
    protected $db;
    protected $table;

    public function __construct(Adapter $db)
    {
        $this->db = $db;
        $this->table = new \Zend\Db\Sql\TableIdentifier('ECOMMERCE_USERS', 'APIGILITY');
    }

    public function fetchAll()
    {
        $select = new Select($this->table);
        $paginatorAdapter = new LowercaseDbSelect($select, $this->db);
        $collection = new EcommerceUserCollection($paginatorAdapter);
        return $collection;
    }

    public function fetchOne($userId)
    {
        $sql = 'SELECT * FROM APIGILITY.ECOMMERCE_USERS WHERE id = ?';
        $resultset = $this->db->query($sql, array($userId));
        $data = $resultset->toArray();
        if (!$data) {
            return false;
        }

        $entity = new EcommerceUserEntity();
        $entity->populate($data[0]);
        return $entity;
    }

    public function save($data, $id = 0)
    {
        $data = (array)$data;
        $parameters = [
            $data['username'],
            $data['email'],
            $data['first_name'],
            $data['last_name'],
        ];
        if ($id > 0) {
            $data['id'] = $id;
            array_push($parameters, $id);
        }


        if (isset($data['id'])) {
            $sql = <<<SQL
UPDATE APIGILITY.ECOMMERCE_USERS
SET USERNAME   = ?,
    EMAIL      = ?,
    FIRST_NAME = ?,
    LAST_NAME  = ?
WHERE ID = ?
SQL;
            $result = $this->db->query($sql, $parameters);
        } else {
            $sql = <<<SQL
INSERT INTO APIGILITY.ECOMMERCE_USERS (USERNAME, EMAIL, FIRST_NAME, LAST_NAME)
VALUES (?, ?, ?, ?)
SQL;

            $result = $this->db->query($sql, $parameters);
            $data['id'] = $this->db->getDriver()->getLastGeneratedValue();
        }

        return $this->fetchOne($data['id']);
    }

    public function delete($id)
    {
        $QQ = new \Zend\Db\Sql\Expression('?');
        $delete = new Delete($this->table);
        $delete->where(['ID' => $QQ]);
        $result = $this->db->query($delete->getSqlString(), [$id]);

        return $result->getAffectedRows() > 0;
    }
}
```

Don't forget to load the Mapper within the Module. Without putting it here, the service will not be accessible
via the service manager. This is necessary for the dependency injection in the resource factory. Just add
this `getServiceConfig()` method to `module/Test/Module.php`.

```php
<?php
// module/Test/Module.php
namespace Test;

use ZF\Apigility\Provider\ApigilityProviderInterface;

class Module implements ApigilityProviderInterface
{
    public function getConfig()
    {
        return include __DIR__ . '/config/module.config.php';
    }

    public function getAutoloaderConfig()
    {
        return [
            'ZF\Apigility\Autoloader' => [
                'namespaces' => [
                    __NAMESPACE__ => __DIR__ . '/src',
                ],
            ],
        ];
    }

    public function getServiceConfig()
    {
        return array(
            'factories' => array(
                'Test\V1\Rest\EcommerceUser\EcommerceUserMapper' =>  function ($sm) {
                    $adapter = $sm->get('ibmdb');
                    return new \Test\V1\Rest\EcommerceUser\EcommerceUserMapper($adapter);
                },
            ),
        );
    }
}

```

> Keep in mind that the name (associative key) for this mapper could be anything. I keep the class name to simply
guarantee this service name will be unique, but one could name it anything for simplification. This is the named used
to pass it to the resource in the factory. For example, it could be:

```php
// Service name defined through associative array key
'foo' =>  function ($sm) {
    $adapter = $sm->get('ibmdb');
    return new \Test\V1\Rest\EcommerceUser\EcommerceUserMapper($adapter);
},
```

### Pass the Mapper to the Resource

The API user accesses the data through the Resource via an HTTP call. How does the Resource access the data, though?
That's exactly the reason we made the mapper. Now we'll inject it to the Resource via the Resrouce Factory. The Resrouce
Factory is there for us to perform dependency injection, so let's do just that.

```php
<?php
// module/Test/src/V1/Rest/EcommerceUser/EcommerceUserResourceFactory.php
namespace Test\V1\Rest\EcommerceUser;

class EcommerceUserResourceFactory
{
    public function __invoke($services)
    {
        $mapper = $services->get('Test\V1\Rest\EcommerceUser\EcommerceUserMapper');
        return new EcommerceUserResource($mapper);
    }
}
```

### Setup the Resource

Since we are injecting a service to the resource, we will need to define that service as a property and initiate it in
the constructor. We'll then use the Mapper throughout. Our Resource ends up looking like:

```php
<?php
// module/Test/src/V1/Rest/EcommerceUser/EcommerceUserResource.php
namespace Test\V1\Rest\EcommerceUser;

use ZF\ApiProblem\ApiProblem;
use ZF\Rest\AbstractResourceListener;

class EcommerceUserResource extends AbstractResourceListener
{
    /**
     * @var \Test\V1\Rest\EcommerceUser\EcommerceUserMapper $mapper
     */
    protected $mapper;

    public function __construct($mapper) {
        $this->mapper = $mapper;
    }

    /**
     * Create a resource
     *
     * @param  mixed $data
     * @return ApiProblem|mixed
     */
    public function create($data)
    {
        return $this->mapper->save($data);
    }

    /**
     * Delete a resource
     *
     * @param  mixed $id
     * @return ApiProblem|mixed
     */
    public function delete($id)
    {
        return $this->mapper->delete($id);
    }

    /**
     * Delete a collection, or members of a collection
     *
     * @param  mixed $data
     * @return ApiProblem|mixed
     */
    public function deleteList($data)
    {
        return new ApiProblem(405, 'The DELETE method has not been defined for collections');
    }

    /**
     * Fetch a resource
     *
     * @param  mixed $id
     * @return ApiProblem|mixed
     */
    public function fetch($id)
    {
        return $this->mapper->fetchOne($id);
    }

    /**
     * Fetch all or a subset of resources
     *
     * @param  array $params
     * @return ApiProblem|mixed
     */
    public function fetchAll($params = [])
    {
        return $this->mapper->fetchAll();
    }

    /**
     * Patch (partial in-place update) a resource
     *
     * @param  mixed $id
     * @param  mixed $data
     * @return ApiProblem|mixed
     */
    public function patch($id, $data)
    {
        return new ApiProblem(405, 'The PATCH method has not been defined for individual resources');
    }

    /**
     * Patch (partial in-place update) a collection or members of a collection
     *
     * @param  mixed $data
     * @return ApiProblem|mixed
     */
    public function patchList($data)
    {
        return new ApiProblem(405, 'The PATCH method has not been defined for collections');
    }

    /**
     * Replace a collection or members of a collection
     *
     * @param  mixed $data
     * @return ApiProblem|mixed
     */
    public function replaceList($data)
    {
        return new ApiProblem(405, 'The PUT method has not been defined for collections');
    }

    /**
     * Update a resource
     *
     * @param  mixed $id
     * @param  mixed $data
     * @return ApiProblem|mixed
     */
    public function update($id, $data)
    {
        return $this->mapper->save($data, $id);
    }
}
```

### Why Do All This?

We could just do all our work directly in the resource, so why go through all this trouble? That's a good question.
In fact, for quite a long time, I did just that. It's fairly simple to just put all the SQL directly in the resource,
manipulate the data there, and return it to the user.

This does work just fine, but what if we want to access our data elsewhere within the API? Say we want to create an RPC
that relies on a lot of different data sources. We might have an RPC that needs to access an ecommerce user, their cart,
their settings, and all sorts of other data sources. Our resource is for accessing the data through an HTTP call. It
expects `REQUEST` data, and we would have to emulate that somehow if we try to access the data via the resource within
the API. So how do we access our data within the API itself?

That's where entities and their mappers come into play. We can access our data and manipulate it all we want without
having to go through the resource. Not only that, but we have entities, an object representation of our data that is
much more fun to handle than raw data.

### Github Source

The entire Apigility example for this tutorial can be found
[here](https://github.com/jbh/apigility-ibm-i){:target="blank"}.
