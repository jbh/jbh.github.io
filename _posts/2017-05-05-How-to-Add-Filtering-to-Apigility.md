---
layout: post
title: How to Add Filtering to Apigility
excerpt: |
    A guide for adding filtering and querying to Apigility. Specifically, adding filtering, sometimes called querying,
    to the Mapper used to serve up Entity and Collection objects.
image: /images/apigility-entity-mapper-tips/new-ecommerce-api-service.png
categories:
    - apigility
    - ibm i
    - db2
---

* TOC
{:toc}

### Setting Up Apigility

This guide will be using the same project setup during my
[Apigility Entity & Mapper Tips](/Apigility-Entity-Mapper-Tips/) article. The related
[github repo](https://github.com/jbh/apigility-ibm-i/){:target="_blank"} has been updated with the new source in this article.

### Create a Filters Class

I found the best way to stay organized was to create a type of Helper Class called `<ServiceName>Filters`. This allows
all filtering where-clauses to be kept in one place and used as needed. A default one that I generate would look
something like:

```php
<?php
// module/Test/src/V1/Rest/EcommerceUser/EcommerceUserFilters.php
namespace Test\V1\Rest\EcommerceUser;

use Zend\Db\Sql\Where;

class EcommerceUserFilters
{
    public static function defaultFilter($filters = [])
    {
        return function (Where $where) use ($filters) {
            foreach ($filters as $filter => $value) {
                $where->expression("{$filter} = ?", $value);
            }
        };
    }

    public static function idFilter($id)
    {
        return function (Where $where) use ($id) {
            $where->expression('ID = ?', $id);
        };
    }

    public static function usersWithUsernameLike($username)
    {
        return function (Where $where) use ($username) {
            $where->expression('USERNAME LIKE ?', "%$username%");
        };
    }
}
```

A basic `Filters` class, like above, should have a `defaultFilter` and `idFilter`. The default filter puts all fields
together in a where clause with `AND`. The id filter simply adds a where clause for the ID, allowing one to select
an individual row. For the sake of having a custom filter example, I've added the method `usersWithUsernameLike`.

### Use the Fitlers Class

Now that we have some filters to play around with, we need use the Filters Class in our Mapper. Anywhere you need a
where clause, simply add it to Filters, and call it in the Mapper.

Here I've added it to the fetch all in order to have the ability to match data in columns:

```php
<?php
namespace Test\V1\Rest\EcommerceUser;

use Zend\Db\Sql\Sql;
use Zend\Db\Sql\Select;
use Zend\Db\Sql\Insert;
use Zend\Db\Sql\Update;
use Zend\Db\Sql\Delete;
use Zend\Db\Sql\TableIdentifier;
use Zend\Db\Sql\Predicate\Expression;
use Zend\Db\Adapter\Adapter;
use SA\Adapter\LowercaseDbSelect;

class EcommerceUserMapper
{
    protected $db;
    protected $sql;
    protected $table;

    public function __construct(Adapter $db)
    {
        $this->db = $db;
        $this->sql = new Sql($db);
        $this->table = new TableIdentifier('ECOMMERCE_USERS', 'APIGILITY');
    }

    public function fetchAll($filters = [])
    {
        $select = empty($filters)                                         // If there are no filters
            ? $this->getNewSimpleSelect()                                 // Get a new simple select
            : $this->applyFilters($this->getNewSimpleSelect(), $filters); // Else apply filters to a new simple select

        $paginatorAdapter = new LowercaseDbSelect($select, $this->db);
        $collection = new EcommerceUserCollection($paginatorAdapter);
        return $collection;
    }

    /**
     * Used to keep column selection consistent. Handy when using data methods like TRIM()
     * Using TRIM as an example here although it is unnecessary in this context.
     *
     * @return array
     */
    private function getColumnsForSelect()
    {
        return [
            'ID'          => 'ID',
            'USERNAME'    => new Expression('TRIM(USERNAME)'),
            'EMAIL'       => 'EMAIL',
            'FIRST_NAME'  => 'FIRST_NAME',
            'LAST_NAME'   => 'LAST_NAME',
            'CREATED_AT'  => 'CREATED_AT',
            'MODIFIED_AT' => 'MODIFIED_AT',
        ];
    }

    private function getNewSimpleSelect()
    {
        $select = new Select($this->table);
        $select->columns($this->getColumnsForSelect());

        return $select;
    }

    private function applyFilters(Select $select, $filters)
    {
        $keys = array_keys((array)$filters);
        $firstKey = count($keys) > 0 ? $keys[0] : false;

        if (array_key_exists('sort', $filters)) {
            // Example sort: ?sort=username, email DESC
            $select->order(new Expression($filters['sort']));
            unset($filters['sort']);
        }

        $filtersClass = 'Test\V1\Rest\EcommerceUser\EcommerceUserFilters';

        $isCustomFilter = $firstKey
            && $firstKey !== 'defaultFilter'
            && method_exists($filtersClass, $firstKey);

        // Example custom filter call: ?usersWithUsernameLike=elean would return the user with the username `eleanor`
        $where = $isCustomFilter                                                  // If this is a custom filter
            ? call_user_func("{$filtersClass}::{$firstKey}", $filters[$firstKey]) // Apply the custom filter
            : EcommerceUserFilters::defaultFilter($filters);                      // Else apply default filter

        $select->where($where);

        return $select;
    }
}
```

> For the above to work, one needs to pass the `$params` array from the Resource to the Mapper fetchAll method.

It's that simple, really. Just make sure to put any filter you want into the `collection_query_whitelist` either through
the Apigility Admin or the module config. Please see the
[github repo](https://github.com/jbh/apigility-ibm-i/){:target="_blank"} for the complete example.

### Disclaimer

I'm not sure if this is the most graceful way of accomplishing this sort of filtering. I do agree that when queries get
complex, one should make an RPC, but what about simple data querying? I don't want to need to build an RPC just to sort
data, for example. This is the solution I came up with after failing to find any good, in-depth examples on data
filtering/querying with Apigility.
