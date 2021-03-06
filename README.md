Yii Sphinx Component
====================

Simple and powerful component for work with Sphinx search engine.

This is a beta version, please help with testing and bug reports.
You can find old stable version at "releases" page.

Features
--------

* Simple query methods
* Extended ESphinxSearchCriteria for complex queries
* Support connection by Sphinx API and Sphinx QL
* Support packeted queries for both connections
* Unit tests coverage


How to install
--------------

###1. via composer

```javascript
"repositories": [
   {
    "type": "vcs",
    "url": "https://github.com/sergebezborodov/sphinx-yii"
   }
],
"require": {
    "sergebezborodov/sphinx-yii": "dev-master"
}
```

in config (assumes you have 'vendor' alias to composer directory):

```php
'components' => array(
    'sphinx' => array(
        'class' => 'vendor.sergebezborodov.sphinx-yii.ESphinxApiConnection', // sphinx api mode
        //'class' => 'vendor.sergebezborodov.sphinx-yii.ESphinxMysqlConnection', for sphinx ql mode
        'server' => array('localhost', 3386),
        'connectionTimeout' => 3, // optional, default 0 - no limit
        'queryTimeout'      => 5, // optional, default 0 - no limit
    ),
),
```


###2. old school way

Download and extract source in protected/extensions folder. In config add:

```php
'import' => array(
    // i hope remove this in new versions
    'ext.sphinx.*',
    'ext.sphinx.ql.*',
    'ext.sphinx.enums.*',
),

'components' => array(
    'sphinx' => array(
        'class' => 'ext.sphinx.ESphinxApiConnection', // sphinx api mode
        //'class' => 'ext.sphinx.ESphinxMysqlConnection', for sphinx ql mode
        'server' => array('localhost', 3386),
        'connectionTimeout' => 3, // optional, default 0 - no limit
        'queryTimeout'      => 5, // optional, default 0 - no limit
    ),
),
```


How to use
----------

All component classes names begins with ESphinx.
Main object we used for querying is ESphinxQuery.

Query in index:
```php
Yii::app()->sphinx->executeQuery(new ESphinxQuery('Hello world!'), 'blog');
```


Extended queries
----------------
Often we need search in index with some parametrs and options. For this task component has class ESphinxSearchCriteria.
It's very similar to CDbCriteria and has the same idea.

Search in article index with some parametrs:

```php
$criteria = new ESphinxSearchCriteria(array(
    'sortMode' => ESphinxSort::EXTENDED,
    'orders' => array(
        'date_created' => 'DESC',
        'date_updated' => 'ASC',
    ),
    'mathMode' => ESphinxMatch::EXTENDED,
));

$query = new ESphinxQuery('@(title,body) hello world', 'articles', $criteria);
```

Criteria can changing at work.

```php
$criteria = new ESphinxSearchCriteria(array('mathMode' => ESphinxMatch::EXTENDED));
$criteria->addFilter('user_id', 1000); // add filter by user, we can use integer or integer array
$criteria->addFilter('site_id', 123, false, 'site'); // add filter by site_id field with key value (will used later)

// querying
$result = Yii::app()->sphinx->executeQuery(new ESphinxQuery('', 'products', $criteria));

// search same query by another site
$criteria->addFilter('site_id', 321, false, 'site'); // change site_id param value

// querying
$result = Yii::app()->sphinx->executeQuery(new ESphinxQuery('', 'products', $criteria));

// search same query but without site_id param
$criteria->deleteFilter('site'); // delete filter on site_id field

// querying....
```


Multi queries
-------------
One of the powerfull sphinx features is multi queries (packet queries). When you send two or more queries
sphinx does internal optimisation for faster work.

```php
$query1 = new ESphinxQuery('', 'products', array('filters' => array(array('site_id', 123))));
$query2 = new ESphinxQuery('', 'products', array('filters' => array(array('site_id', 321))));

$results = Yii::app()->sphinx->executeQueries(array($query1, $query2));
```


Another way to add queries:

```php
$query = new ESphinxQuery('', 'products', array('filters' => array(array('site_id', 123, 'key' => 'site_id')))));
Yii::app()->sphinx->addQuery($query);

// change previous site_id filter value
$query->criteria->addFilter('site_id', 321, false, 'site_id');

$results = Yii::app()->sphinx->runQueries();
```

Options
-------

####Versions
ESphinxSearchCriteria includes all possible options of last sphinx beta.
Be sure you are using right functions for your version.

####Sort Methods
For SPH_SORT_EXTENDED (ESphinxSort::EXTENDED) you should use setOrders() or addOrder() method.
For others sort modes use setSortBy() for one field.

Using The DataProvider
----------------------
A simpler way to search is to use the ESphinxDataProvider to automatically fetch the models for your search results.

Example searching Post model:
```php
  $dataProvider=new ESphinxDataProvider('Post', array(
      'query'=>'@title learn php',
      'sphinxCriteria' => new ESphinxCriteria(array(
         'matchMode' => ESphinxMatch::EXTENDED,
       )),
      'criteria'=>array(
         'condition'=>'author.name = :author',
         'params'=>array(':author'=>'Mark O\'Keeffe'),
         'order'=>'create_time DESC',
         'with'=>array('author'),
      ),
      'pagination'=>array(
         'pageSize'=>20,
      ),
  ));
```

Then use the dataprovider as normal