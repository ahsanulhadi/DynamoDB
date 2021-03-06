DynamoDB by Ajay Deshpande.

============================================================================================================
Create object of class.
============================================================================================================

$dynamodb = new dynamoDBInput($AWS_KEY,$AWS_SECRET,$AWS_REGION);


============================================================================================================
Creating a table.
============================================================================================================

Set Read And Write Units:
========================

$dynamodb->setUnits($read,$write);

==>Read : Provisioned read units. Default: 10
==>Write : Provisioned write units. Default: 5 
ex: $dynamodb->setUnits(10,5);



Create Table: 
============================================================================================================

$dynamodb->createTable('TableName, PrimaryIndex : X as T of Hash;Y as T of Range',' LocalSecondaryIndex : { X as T of Range;Y as T of Hash | IndexName | ProjectionType | NonKeyAttributes } { X as T of Range;Y as T of Hash | IndexName | ProjectionType | NonKeyAttributes}','Read','Write');


Ex:
$dynamodb -> createTable('Orders, PrimaryIndex : CustomerID as N of HASH;OrderID as S of RANGE, LocalSecondaryIndex : CustomerID as N of HASH;OrderDate as S of RANGE | OrderDateIndex | KEYS_ONLY ');

==>TableName - Name of the table

==>T can be : 
S - (string) - Represents a String data type
N - (string) - Represents a Number data type
B - (string) - Represents a Binary data type
SS - (array[string]) Array of string values - Represents a String set data type
NS - (array[string]) Array of string values - Represents a Number set data type
BS - (array[string]) Array of string values - Represents a Binary set data type

==> Set Hash for hash index and Range for range index.

==>LocalSecondaryIndex : Set one or more local secondary indexes defined in {} brackets.

==>IndexName - The name of the secondary index. Must be unique only for this table.
==>Projection - (hash[string => value]) Associative array.
=====>ProjectionType - [one of ALL, KEYS_ONLY, INCLUDE]  - Represents the set of attributes that are projected into the index:
=====>NonKeyAttributes - Array of string values - Represents the non-key attribute names which will be projected into the index.




============================================================================================================
Inserting a record.
============================================================================================================

$dynamodb->insert('insert into TableName : Key => Value ; Key => Value');

Ex:
$dynamodb->insert('insert into Orders : CustomerID => 2 ; OrderID => 1 ; OrderDate => 30/06/2013 ; Remarks => Excellent');



============================================================================================================
Selecting records.
============================================================================================================

/
Set Select parameter:
=====================
$dynamodb->setSelect($value);
==>Select value can be: ALL_ATTRIBUTES, ALL_PROJECTED_ATTRIBUTES, SPECIFIC_ATTRIBUTES, COUNT .


Set index Name : 
================
$dynamodb->setIndexName($value);

==>Set IndexName to query on a local secondary index.


Get values: 
============================================================================================================


$res = $dynamodb-> query('TableName : key Op Value ; Key Op Value ',$limit);
                                        ========OR=========
                                         For Where clause : 
                                        ===================
$res = $dynamodb->query( 'TableName : where ( ( key Op Value  ) and ( key Op Value ) ) or ( key Op Value )',$limit);
$res = $dynamodb->scan( 'TableName : where ( ( key Op Value  ) and ( key Op Value ) ) or ( key Op Value )',$limit);

====================================================================================================================
====================================================================================================================
====================================================================================================================
IMP : Each of the conditions must be enclosed in parantheses. The above query should not be given as:
        $res = $dynamodb->query( 'TableName : where ( key Op Value and key Op Value  ) or ( key Op Value )',$limit);
                                                    OR.
        $res = $dynamodb->scan( 'TableName : where ( key Op Value and key Op Value  ) or ( key Op Value )',$limit);
====================================================================================================================
====================================================================================================================
====================================================================================================================


==>Key   : Key to be set.
==>Value : Contains exactly one value, except for a BETWEEN or IN comparison, in which case it contains two values.
==>Op    :  EQ, NE, IN, LE, LT, GE, GT, BETWEEN, NOT_NULL, NULL, CONTAINS, NOT_CONTAINS, BEGINS_WITH.
==>$limit is the number of items in response.


Select,IndexName,limit ==> optional.


Get item: 
=========

$res = $dynamodb->query( 'errors : id EQ 1201 ; time GT -15 Days');

With Where clause :
===================

$res = $dynamodb->query( 'errors : where ( ( id EQ 1201 ) and ( time GT -15 Days ) ) and ( id EQ 2003 ) ');
$res = $dynamodb->scan( 'errors : where ( ( id EQ 1201 ) and ( time GT -15 Days ) ) or ( id EQ 2003 ) ');

With Limit:
============


res = $dynamodb->query( 'errors : id EQ 1201 ; time GT -15 Days',$limit);

or

$res = $dynamodb->query( 'errors : where ( ( id EQ 1201 ) and ( time GT -15 Days ) ) and ( id EQ 2003 ) ',$limit);

Select single row:
===================

select('TableName : HashKey => Value ;  RangeKey => Value');

OR : 

Use above query with limit 1.



Order results:
==============

orderBy($res, Attribute,Order);

==>Attribute: The attribute to sort the values on.
==>Order: either 'asc' or 'desc'.


Ex:

$res = $dynamodb->orderBy($res, 'id','asc');


Count records :
=============

==> To count records set 'Select' in the query to COUNT.
Ex: 
$dynamodb->setSelect('Count');




============================================================================================================
Deleting a record:
============================================================================================================

delete('tableName : Key => Value ; Key => Value')

Ex:

$dynamodb->delete('errors: id => 2023 ; time => 1372614625');

============================================================================================================
Updating a record:
============================================================================================================

update('tableName','key => value ; key => value'//Keys,'key => Value => Action ; key => Value => Action'//UpdateAttrs)

All parameters necessary.

ex : 

$dynamodb->update('thread','ForumName => AmazonDB ; Subject => Number','LastPostedBy => asd@asd.com => PUT');



=============================================================================================================
P.S, regarding the Where clause in select.
=============================================================================================================

Both the query and scan operations can be used to get rows. 

The query operation works only when the search is done using a hash key. 

The scan operation works on any attribute, but it scans the whole table and returns upto a limit of 1MB.

So if the select is on a hash key, query is efficient, whereas if it is on a non hash key use scan.




Example :::: 
$dynamodb = new dynamoDBInput($AWS_KEY, $AWS_SECRET, $AWS_REGION);

$dynamodb ->setUnits(10, 5);

$dynamodb -> createTable('Orders, PrimaryIndex : CustomerID as N of HASH;OrderID as N of RANGE','LocalSecondaryIndex : { CustomerID as N of HASH;OrderDate as N of RANGE | OrderDateIndexs | INCLUDE | [ CustomerID , OrderID ]}, { CustomerID as N of HASH;OrderDate as N of RANGE | OrderDateIndexes | KEYS_ONLY } ');

$dynamodb->insert('insert into Orders : CustomerID => 3 ; OrderID => 3 ; OrderDate => 25/06/2013 ; Remarks => Good');
$dynamodb->insert('insert into Orders : CustomerID => 4 ; OrderID => 4 ; OrderDate => 1/07/2013 ; Remarks => Good');
$res = $dynamodb->query( 'Orders : CustomerID EQ 4 ; OrderID GT 2');
print_r($res);


//Select by limit. For single row set $limit to 1
$dynamodb->query( 'Orders : CustomerID EQ 4',1);

$dynamodb->delete('Orders: CustomerID => 4; OrderID => 4');

$dynamodb->insert('insert into Orders : CustomerID => 2 ; OrderID => 2 ; OrderDate => 1/07/2013 ; Remarks => Good');

$dynamodb->update('Orders',' CustomerID => 2 ; OrderID => 2 ','LastPostedBy => asd@asd.com => PUT');

$dynamodb->setSelect('COUNT');
$res = $dynamodb->scan( 'Orders : where ( CustomerID EQ 2 )  and ( ( CustomerID EQ 3 ) and ( OrderID EQ 3) )');
print_r($dynamodb->orderBy($res, 'CustomerID','desc'));
