# 1. List containers

## 1.1 Return a list of containers

```php
$containerList = $service->listContainers();

while ($container = $containerList->next()) {
    // ... do stuff
}
```

Container names are sorted based on a binary comparison, a single built-in collating sequence that compares string
data using SQLite's memcmp() function, regardless of text encoding.

The list is limited to 10,000 containers at a time. See 1.3 for ways to limit and navigate this list.

## 1.2 Return a formatted list of containers

Currently, the SDK only supports JSON-formatted responses.

## 1.3 Controlling a large list of containers

You may limit and control this list of results by using the `marker` and `end_marker` parameters. The former parameter
(`marker`) tells the API where to begin the list, and the latter (`end_marker`) tells it where to end the list. You may
use either of them independently or together. You may also use the `limit` parameter to fix the number of containers
returned.

To list a set of containers between two fixed points:

```php
$someContainers = $service->listContainers(array(
    'marker'     => 'container_55',
    'end_marker' => 'container_2001'
));
```

Or to return a limited set:

```php
$someContainers = $service->listContainers(array('limit' => 560));
```

# Get container

To retrieve a certain container, either to access its object or metadata:

```php
$container = $service->getContainer('container_name');

echo $container->getObjectCount();
echo $container->getBytesUsed();
```

# Create container

To create a new container, you just need to define its name:

```php
$container = $service->createContainer('my_amazing_container');
```

If the response returned is `FALSE`, there was an API error - most likely due to the fact you have a naming collision.

Container names must be valid strings between 0 and 256 characters. Forward slashes are not currently permitted.

# Delete container

Deleting a container is easy:
```php
$container->delete();
```

Please bear mind that you must delete all objects inside a container before deleting it. This is done for you if you
set the `$deleteObjects` parameter to `TRUE`. You can also do it manually:

```php
$container->deleteAllObjects();
```

# Create or update container metadata

```php
$container->saveMetadata(array(
    'Author' => 'Virginia Woolf',
    'Published' => '1931'
));
```

Please bear in mind that this action will set metadata to this array - overriding existing values and wiping those left
out. To _append_ values to the current metadata:

```php
$metadata = $container->appendToMetadata(array(
    'Publisher' => 'Hogarth'
));
```

If you only want to set the metadata to the local object, and not immediately retain these values on the API, you can
use a standard setter method - which can contribute to eventual actions like an update:

```php
$container->setMetadata(array('Foo' => 'Bar'));
```

# Container quotas

The container_quotas middleware implements simple quotas that can be imposed on Cloud Files containers by a user.
Setting container quotas can be useful for limiting the scope of containers that are delegated to non-admin users,
exposed to formpost uploads, or just as a self-imposed sanity check.

To set quotas for a container:

```php
use OpenCloud\Common\Constants\Size;

$container->setCountQuota(1000);
$container->setBytesQuota(2.5 * Size::GB);
```

And to retrieve them:

```php
echo $container->getCountQuota();
echo $container->getBytesQuota();
```

# Access log delivery

To view your object access, turn on Access Log Delivery. You can use access logs to analyze the number of people who
access your objects, where they come from, how many requests for each object you receive, and time-based usage patterns
(such as monthly or seasonal usage).

```php
$container->enableLogging();
$container->disableLogging();
```