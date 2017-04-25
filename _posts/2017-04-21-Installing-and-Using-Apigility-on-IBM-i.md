---
layout: post
title: Installing and Using Apigility on IBM i
excerpt: Using Apigility on the IBM i to access DB2 data.
categories:
    - ibm i
    - apigility
    - db2
---

* TOC
{:toc}

### Install Apigility

I find the easiest way to accomplish this on the IBM i is to use php itself on command line:

{% highlight bash %}
$ php -r "readfile('https://apigility.org/install');" | php
{% endhighlight %}

After this is ran, Apigility will try to serve itself up. While it is successful in doing so, I have not been able
to visit Apigility without errors when it is running this way on the IBM i. Therefore, I set it up normally through
apache:

{% highlight apache %}
<VirtualHost *:80>
    ServerName api.ibmiserver.com
    DocumentRoot /path/to/apigility/public

    <Directory "/path/to/apigility/public">
        Options FollowSymLinks
        order allow,deny
        allow from all
        AllowOverride all
    </Directory>

    AllowEncodedSlashes On
</VirtualHost>
{% endhighlight %}

That should do it! Apigility should be running fine at the address provided as ServerName.

### Create a DB2 database adapter

Click on `Database` in the upper navigation, then click `New DB Adapter` in the content area.
Fill out the necessary fields and click `Save`.

![New Database Adapter Screenshot]({{site.url}}/images/new-database-adapter.png)

The adapter options can be added and edited through the interface as well:

![Edit Database Driver Options Screenshot]({{site.url}}/images/edit-database-driver-options.png)

If left with no options set, then we are limited to using the `dot` syntax, (`library.file`) only. All column names
will be uppercase as well. Some helpful options to set in `local.php`:

{% highlight php %}
<?php
// config/autoload/local.php
return [
    'db' => [
        'adapters' => [
            'ibmdb' => [
                'database' => 'SXXXXXXX',
                'driver' => 'IbmDb2',
                'username' => 'user',
                'password' => 'password',
                'driver_options' => [
                    'i5_naming' => 1, // This allows us to write with slashes as well. LIBRARY/FILE
                    'db2_attr_case' => 1, // This forces all returned column names to lowercase. Personal preference.
                    // Other helpful options can be found at: http://php.net/manual/en/function.db2-connect.php
                ],
            ]
        ],
    ],
];
{% endhighlight %}

### Create an API and a Service

This part is fairly straightforward. Either follow the official documentation for creating rest services, or create
a simple REST service for test purposes, which is what I'm going to do here.

![Test API Example Screenshot]({{site.url}}/images/test-api-example.png)

I created an API called Test with a REST service called Test. All the Test service does is return all PTFs with
updates available when a GET request is sent for a collection.

So if we call `http://api.ibmiserver.com/test` through a service like Postman, we should get something like:

{% highlight json %}
{
  "_links": {
    "self": {
      "href": "http://test.api.t-r.com/test"
    }
  },
  "_embedded": {
    "test": [
      {
        "ptf_group_currency": "PSP INFORMATION NOT AVAILABLE",
        "ptf_group_id": "SF99145",
        "ptf_group_title": "PERFORMANCE TOOLS",
        "ptf_group_level_installed": 9,
        "ptf_group_level_available": null,
        "ptf_group_last_updated_by_ibm": null,
        "ptf_group_release": "V7R1M0",
        "ptf_group_status_on_system": "INSTALLED"
      },
      {
        "ptf_group_currency": "PSP INFORMATION NOT AVAILABLE",
        "ptf_group_id": "SF99362",
        "ptf_group_title": "BACKUP RECOVERY SOLUTIONS",
        "ptf_group_level_installed": 54,
        "ptf_group_level_available": null,
        "ptf_group_last_updated_by_ibm": null,
        "ptf_group_release": "V7R1M0",
        "ptf_group_status_on_system": "INSTALLED"
      }
    ]
  },
  "total_items": 2
}
{% endhighlight %}

### Poor Man's REST

Instead of relying Entity and Collection definitions, which would be the proper way, I have used a barebones approach
to creating RESTful services when on the IBM i. This is largely due to the data I deal with daily, which is
non-normalized and has column names that sometimes use special characters. Let's take a look at the
Resource and ResourceFactory for this service in case others need to take this same approach.

{% highlight php %}
<?php
// module/Test/src/V1/Rest/Test/TestResourceFactory.php
namespace Test\V1\Rest\Test;

class TestResourceFactory
{
    /**
     * This is so PHPStorm knows what type of Object $services is.
     * @param \Zend\ServiceManager\ServiceManager $services
     * @return TestResource
     */
    public function __invoke($services)
    {
        // Grab the ibmdb service we created earlier to define our database
        $db = $services->get('ibmdb');

        // Inject that database into TestResource for use in the Test REST service.
        return new TestResource($db);
    }
}
{% endhighlight %}

{% highlight php %}
<?php
// module/Test/src/V1/Rest/Test/TestResource.php
namespace Test\V1\Rest\Test;

use ZF\ApiProblem\ApiProblem;
use ZF\Rest\AbstractResourceListener;

class TestResource extends AbstractResourceListener
{
    /**
     * Define $db variable
     * @var \Zend\Db\Adapter\Adapter $db
     */
    private $db;

    public function __construct($db) {
        // Initiate $db variable with what is injected from the factory
        $this->db = $db;
    }

    /**
     * Fetch all or a subset of resources
     *
     * @param  array $params
     * @return ApiProblem|mixed
     */
    public function fetchAll($params = [])
    {
        $sql = <<<SQL
SELECT * FROM SYSTOOLS.GROUP_PTF_CURRENCY
ORDER BY PTF_GROUP_LEVEL_AVAILABLE - PTF_GROUP_LEVEL_INSTALLED DESC
SQL;

        $resultSet = $this->db->query($sql, []);

        return $resultSet->toArray();
    }
}

{% endhighlight %}

While it isn't ideal to be running raw sql in the Resource, sometimes we don't have much choice, and I wanted
to demonstrate a simple way to get started on the IBM i. That's it for now. Stay tuned for
Apigility + OAuth2 on the IBM i and how to digest these services with Angular2.
