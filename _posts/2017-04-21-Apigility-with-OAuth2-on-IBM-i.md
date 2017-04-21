---
layout: post
title: Apigility with OAuth2 on IBM i
excerpt: Using Apigility with OAuth2 on IBM i utilizing DB2 as the data source.
categories:
    - ibm i
    - apigility
    - db2
---

* TOC
{:toc}

### Create the OAuth tables in DB2

These tables are created according to specifications from [Zend Framework OAuth2](https://github.com/zfcampus/zf-oauth2).

{% highlight sql %}
-- Replace LIBRARY with the preferred library name.
-- Build OAuth tables for Apigility.
--   OAUTH_CLIENTS
--   OATUH_USERS
--   OAUTH_ACCESS_TOKENS
--   OAUTH_REFRESH_TOKENS
--   OAUTH_CODES
--   OAUTH_SCOPES
--   OAUTH_JWT
-- More details at https://github.com/zfcampus/zf-oauth2


-- BEGIN OAUTH_CLIENTS DEFINITION
CREATE OR REPLACE TABLE LIBRARY/OAUTH_CLIENTS (
  CLIENT_ID FOR COLUMN CLID VARCHAR(80) NOT NULL,
  CLIENT_SECRET FOR COLUMN CLSCRT VARCHAR(80) NOT NULL,
  REDIRECT_URI FOR COLUMN CLRDURI VARCHAR(2000) NOT NULL,
  GRANT_TYPES FOR COLUMN CLGRNTYPS VARCHAR(80),
  SCOPE FOR COLUMN CLSCOPE VARCHAR(2000),
  USER_ID FOR COLUMN CLUSRID VARCHAR(255),

  PRIMARY KEY (CLIENT_ID)
) ;

RENAME TABLE LIBRARY/OAUTH_CLIENTS
  TO SYSTEM NAME OACLNTS ;

LABEL ON TABLE LIBRARY/OAUTH_CLIENTS IS 'OAUTH CLIENTS' ;

LABEL ON COLUMN LIBRARY/OAUTH_CLIENTS (
  CLID TEXT IS 'CLIENT ID',
  CLSCRT TEXT IS 'CLIENT SECRET',
  CLRDURI TEXT IS 'REDIRECT URI',
  CLGRNTYPS TEXT IS 'GRANT TYPES',
  CLSCOPE TEXT IS 'SCOPE',
  CLUSRID TEXT IS 'USER ID'
) ;
-- END OAUTH_CLIENTS DEFINITION

-- BEGIN OAUTH_USERS DEFINITION
CREATE OR REPLACE TABLE LIBRARY/OAUTH_USERS (
  USERNAME FOR COLUMN UUSRNM VARCHAR(255) NOT NULL,
  PASSWORD FOR COLUMN UPSSWRD VARCHAR(2000),
  USER_TABLE FOR COLUMN UUSRTBL VARCHAR(128) NOT NULL,

  PRIMARY KEY (USERNAME)
) ;

RENAME TABLE LIBRARY/OAUTH_USERS
  TO SYSTEM NAME OAUSERS ;

LABEL ON TABLE LIBRARY/OAUTH_USERS IS 'OAUTH USERS' ;

LABEL ON COLUMN LIBRARY/OAUTH_USERS (
  UUSRNM TEXT IS 'USERNAME',
  UPSSWRD TEXT IS 'PASSWORD',
  UUSRTBL TEXT IS 'EXTERNAL USER TABLE'
) ;
-- END OAUTH_USERS DEFINITION

-- BEGIN OAUTH_ACCESS_TOKENS DEFINITION
CREATE OR REPLACE TABLE LIBRARY/OAUTH_ACCESS_TOKENS (
  ACCESS_TOKEN FOR COLUMN ATTKN VARCHAR(40) NOT NULL,
  CLIENT_ID FOR COLUMN ATCLID VARCHAR(80) NOT NULL,
  USER_ID FOR COLUMN ATUSRID VARCHAR(255),
  EXPIRES FOR COLUMN ATEXPRS VARCHAR(32) NOT NULL,
  SCOPE FOR COLUMN ATSCOPE VARCHAR(2000),

  PRIMARY KEY (ACCESS_TOKEN)
) ;

RENAME TABLE LIBRARY/OAUTH_ACCESS_TOKENS
  TO SYSTEM NAME OAACCTKNS ;

LABEL ON TABLE LIBRARY/OAUTH_ACCESS_TOKENS IS 'OAUTH ACCESS TOKENS' ;

LABEL ON COLUMN LIBRARY/OAUTH_ACCESS_TOKENS (
  ATTKN TEXT IS 'ACCESS TOKEN',
  ATCLID TEXT IS 'CLIENT ID',
  ATUSRID TEXT IS 'USER ID',
  ATEXPRS TEXT IS 'EXPIRES',
  ATSCOPE TEXT IS 'SCOPE'
) ;
-- END OAUTH_ACCESS_TOKENS DEFINITION

-- BEGIN OAUTH_REFRESH_TOKENS DEFINITION
CREATE OR REPLACE TABLE LIBRARY/OAUTH_REFRESH_TOKENS (
  REFRESH_TOKEN FOR COLUMN RTRFTKN VARCHAR(40) NOT NULL,
  CLIENT_ID FOR COLUMN RTCLID VARCHAR(80) NOT NULL,
  USER_ID FOR COLUMN RTUSRID VARCHAR(255),
  EXPIRES FOR COLUMN RTEXPRS VARCHAR(32) NOT NULL,
  SCOPE FOR COLUMN RTSCOPE VARCHAR(2000),

  PRIMARY KEY (REFRESH_TOKEN)
) ;

RENAME TABLE LIBRARY/OAUTH_REFRESH_TOKENS
  TO SYSTEM NAME OARFTKNS ;

LABEL ON TABLE LIBRARY/OAUTH_REFRESH_TOKENS IS 'OAUTH REFRESH TOKENS' ;

LABEL ON COLUMN LIBRARY/OAUTH_REFRESH_TOKENS (
  RTRFTKN TEXT IS 'REFRESH TOKEN',
  RTCLID TEXT IS 'CLIENT ID',
  RTUSRID TEXT IS 'USER ID',
  RTEXPRS TEXT IS 'EXPIRES',
  RTSCOPE TEXT IS 'SCOPE'
) ;
-- END OAUTH_REFRESH_TOKENS DEFINITION

-- BEGIN OAUTH_CODES DEFINITION
CREATE OR REPLACE TABLE LIBRARY/OAUTH_CODES (
  AUTHORIZATION_CODE FOR COLUMN ACAUTHCD VARCHAR(40) NOT NULL,
  CLIENT_ID FOR COLUMN ACCLID VARCHAR(80) NOT NULL,
  USER_ID FOR COLUMN ACUSRID VARCHAR(255),
  REDIRECT_URI FOR COLUMN ACRDURI VARCHAR(2000),
  EXPIRES FOR COLUMN ACEXPRS VARCHAR(32) NOT NULL,
  SCOPE FOR COLUMN ACSCOPE VARCHAR(2000),
  ID_TOKEN FOR COLUMN ACIDTKN VARCHAR(2000),

  PRIMARY KEY (AUTHORIZATION_CODE)
) ;

RENAME TABLE LIBRARY/OAUTH_CODES
  TO SYSTEM NAME OAAUTHCDS ;

LABEL ON TABLE LIBRARY/OAUTH_CODES IS 'OAUTH AUTHORIZATION CODES' ;

LABEL ON COLUMN LIBRARY/OAUTH_CODES (
  ACAUTHCD TEXT IS 'AUTHORIZATION CODE',
  ACCLID TEXT IS 'CLIENT ID',
  ACUSRID TEXT IS 'USER ID',
  ACRDURI TEXT IS 'REDIRECT URI',
  ACEXPRS TEXT IS 'EXPIRES',
  ACSCOPE TEXT IS 'SCOPE',
  ACIDTKN TEXT IS 'ID TOKEN'
) ;
-- END OAUTH_CODES DEFINITION

-- BEGIN OAUTH_SCOPES DEFINITION
CREATE OR REPLACE TABLE LIBRARY/OAUTH_SCOPES (
  TYPE FOR COLUMN SCTYPE VARCHAR(255) NOT NULL DEFAULT 'supported',
  SCOPE FOR COLUMN SCSCOPE VARCHAR(2000),
  CLIENT_ID FOR COLUMN SCCLID VARCHAR(80),
  IS_DEFAULT FOR COLUMN SCISDFLT SMALLINT DEFAULT NULL
) ;

RENAME TABLE LIBRARY/OAUTH_SCOPES
  TO SYSTEM NAME OASCOPES ;

LABEL ON TABLE LIBRARY/OAUTH_SCOPES IS 'OAUTH SCOPES' ;

LABEL ON COLUMN LIBRARY/OAUTH_SCOPES (
  SCTYPE TEXT IS 'TYPE',
  SCSCOPE TEXT IS 'SCOPE',
  SCCLID TEXT IS 'CLIENT_ID',
  SCISDFLT TEXT IS 'IS DEFAULT?'
) ;
-- END OAUTH_SCOPES DEFINITION

-- BEGIN OAUTH_JWT DEFINITION
CREATE OR REPLACE TABLE LIBRARY/OAUTH_JWT (
  CLIENT_ID FOR COLUMN JWTCLID VARCHAR(80) NOT NULL,
  SUBJECT FOR COLUMN JWTSUBJ VARCHAR(80),
  PUBLIC_KEY FOR COLUMN JWTPUBKEY VARCHAR(2000),

  PRIMARY KEY (CLIENT_ID)
) ;

LABEL ON TABLE LIBRARY/OAUTH_JWT IS 'OAUTH JWT' ;

LABEL ON COLUMN LIBRARY/OAUTH_JWT (
  JWTCLID TEXT IS 'CLIENT ID',
  JWTSUBJ TEXT IS 'SUBJECT',
  JWTPUBKEY TEXT IS 'PUBLIC KEY'
) ;
-- END OAUTH_JWT DEFINITION
{% endhighlight %}

### Configure Apigility

> It is important to note that it is best practice to create this Authentication through the Apigility admin, then
edit what is generated.

![New Authentication Adapter Screenshot]({{site.url}}/images/new-authentication-adapter.png)

{% highlight php %}
<?php
// global.php
// Replace library with the appropriate library
return [
    'zf-oauth2' => [
        // Override the default tables in order to prefix with library.
        'storage_settings' => [
            'client_table' => 'library.oauth_clients',
            'access_token_table' => 'library.oauth_access_tokens',
            'refresh_token_table' => 'library.oauth_refresh_tokens',
            'code_table' => 'library.oauth_codes',
            'user_table' => 'library.oauth_users',
            'jwt_table' => 'library.oauth_jwt',
            'scope_table' => 'library.oauth_scopes',
        ]
    ],
    'router' => [
        'routes' => [
            'oauth' => [
                'options' => [
                    'spec' => '%oauth%',
                    'regex' => '(?P<oauth>(/oauth))',
                ],
                'type' => 'regex',
            ],
        ],
    ],
];

{% endhighlight %}

{% highlight php %}
<?php
// local.php
return [
    'zf-mvc-auth' => [
        'authentication' => [
            'adapters' => [
                'oauth-name' => [
                    'adapter' => \ZF\MvcAuth\Authentication\OAuth2Adapter::class,
                    'storage' => [
                        'adapter' => \pdo::class,
                        'dsn' => 'ibm:SXXXXXXX',
                        'route' => '/oauth',
                        'username' => 'user',
                        'password' => 'password',
                    ],
                ],
            ],
        ],
    ],
];

{% endhighlight %}

### Test

That's it! It's that simple to get basic OAuth2 up and running with Apigility on the IBM i. Now one can visit
the [Apgility OAuth2 Documentation](https://apigility.org/documentation/auth/authentication-oauth2) to see how to
connect web server applications up.

One easy way to just quickly test if OAuth2 is working properly is to put a record in OAUTH_CLIENTS, and go
to `/oauth` to test it out.

Encrypt a password

{% highlight bash %}
$ cd /path/to/apigility
$ php vendor/zfcampus/zf-oauth2/bin/bcrypt.php test
{% endhighlight %}

The output for encrypting test should be `$2y$10$8gHQy/sn0vB8H5wbAbhUi.tbUfpf6aE7PBllKHeKaCYTqEyd7vjo6`. Now just fill
in OAUTH_CLIENTS with this.

{% highlight SQL %}
-- Replace LIBRARY with the appropriate library.
-- Insert a new row into OAUTH_CLIENTS for testing purposes.
INSERT INTO LIBRARY.OAUTH_CLIENTS
VALUES (
  'testclient',
  '$2y$10$8gHQy/sn0vB8H5wbAbhUi.tbUfpf6aE7PBllKHeKaCYTqEyd7vjo6',
  '/oauth/receivecode',
  null,
  null,
  null
)
{% endhighlight %}

Now that we have a client record, we should be able to test if we can get an access token. Simply go to

```
http://api.ibmiserver.com/oauth/authorize?response_type=code&client_id=testclient&redirect_uri=/oauth/receivecode&state=xyz
```

If you're able to click `yes` and get an access token, all should be working properly.

### Overriding the Default OAuth2Adapter

For some of us, authentication can be messier than the default. Thankfully, overriding the default factory and
adapter is fairly simple.

First, we need to create a destination folder for custom classes. This can be placed anywhere in the project. I suggest putting it in a
folder in the root of the project. Something like `phplib`. Define a namespace and autoload classes from here in your
`composer.json`:

{% highlight json %}
"autoload": {
    "psr-4": {
        "CompanyNamespace\\": "phplib/"
    }
}
{% endhighlight %}

Now that we have a destination folder created and autoloaded, it's time to create our factory.

{% highlight php %}
<?php
// phplib/Factory/OAuth2AdapterFactory.php
namespace CompanyNamespace\Factory;

use Zend\ServiceManager\ServiceLocatorInterface;
use CompanyNamespace\Adapter\OAuth2Adapter;
use Zend\Db\Adapter\Driver\Pdo\Pdo as PdoDriver;

class OAuth2AdapterFactory
{
    /**
     * Create service
     *
     * @param ServiceLocatorInterface $serviceLocator
     * @return OAuth2Adapter
     */
    public function __invoke(ServiceLocatorInterface $serviceLocator)
    {
        $driver = new PdoDriver(
            new \PDO(
                'ibm:SXXXXXXX',
                'user',
                'password',
                [
                    \PDO::I5_ATTR_DBC_SYS_NAMING => true,
                    \PDO::ATTR_CASE => \PDO::CASE_LOWER
                ]
            )
        );

        if (!$driver instanceof PdoDriver) {
            throw new \RuntimeException("Need a PDO connection!");
        }

        $connection = $driver->getConnection();

        $pdo = $connection->getResource();
        $settings = [
            'client_table' => 'filesrdb.oauth_clients',
            'access_token_table' => 'filesrdb.oauth_access_tokens',
            'refresh_token_table' => 'filesrdb.oauth_refresh_tokens',
            'code_table' => 'filesrdb.oauth_codes',
            'user_table' => 'filesrdb.oauth_users',
            'jwt_table' => 'filesrdb.oauth_jwt',
            'scope_table' => 'filesrdb.oauth_scopes',
        ];

        return new OAuth2Adapter($pdo, $settings);
    }
}

{% endhighlight %}

This factory is used for dependency injection into the custom OAuth2Adapter we're going to build. This will
allow us to override the authentication functionality.

We have a factory, so all we need is an Adapter for it.

{% highlight php %}
<?php
// phplib/Adapter/OAuth2Adapter.php
namespace CompanyNamespace\Adapter;

use ZF\OAuth2\Adapter\PdoAdapter;

/**
 * Custom extension of PdoAdapter to validate against the IBM i system users.
 */
class OAuth2Adapter extends PdoAdapter
{
    /**
     * checkPassword
     *
     * Used for user authentication. $user is an array retrieved with getUser. If the password field
     * ($user['password']) exists, then it will use the normal method of verifyHash to authenticate the user.
     * If the password field does not exist, it will use the db2_connect method to validate if the user is
     * a system user or not.
     *
     * @param array $user
     * @param string $password
     * @return bool|resource
     */
    protected function checkPassword($user, $password)
    {
        // If password exists, verify hash, else test if it is a system user via db connect
        if (array_key_exists('password', $user)) {
            /**
             * Yet another fallback in case the user exists in both the oauth table and the custom_system_user_table table.
             * This happens if an ecommerce user happens to pick the same username as an admin.
             * A better fallback should probably be made. This is a potential security risk
             * if the ecommerce user also happens to have the same password as an admin.
             */
            $isVerified = $this->verifyHash($password, $user['password']);

            return $isVerified ? $isVerified : db2_connect('*LOCAL', $user['user_id'], $password);
        } else {
            return db2_connect('*LOCAL', $user['user_id'], $password);
        }
    }

    /**
     * getUser
     *
     * Simply gets the user's information. At this point, the user is being retrieved in order to test the password
     * with checkPassword. If the user isn't found in the OAUTH_USERS table, it will fallback to the signon table for
     * system user information.
     *
     * @param $username
     * @return array|bool
     */
    public function getUser($username)
    {
        // Prepare and execute SQL for getting the user from the OAUTH_USERS table
        $stmt = $this->db->prepare($sql = sprintf('SELECT * from %s where username=:username', $this->config['user_table']));
        $stmt->execute(array('username' => $username));

        // Try to retrieve the user info. If no info retrieved, fallback to the custom_system_user_table table
        // sa.TODO - figure out how to work with ecuser and signon having same user
        if (!$userInfo = $stmt->fetch(\PDO::FETCH_ASSOC)) {
            // Prepare and execute SQL for getting the user from the SIGNON table
            $stmt = $this->db->prepare($sql = 'SELECT * from library.custom_system_user_table where nuser=:username');
            $stmt->execute(array('username' => $username));

            // Try to retrieve the system user info. If no info retrieved, return false.
            if (!$userInfo = $stmt->fetch(\PDO::FETCH_ASSOC)) {
                return false;
            }
        }

        // the default behavior is to use "username" as the user_id
        return array_merge(array(
            'user_id' => $username
        ), $userInfo);
    }
}
{% endhighlight %}

This particular example is overriding the OAuth2 Adapter in order to also check for system users vs normal users when
someone is authenticating with the API. One can of course do whatever they like in the two validation methods within
the adapter.
