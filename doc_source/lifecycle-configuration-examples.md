# Examples of S3 Lifecycle configuration<a name="lifecycle-configuration-examples"></a>

This section provides examples of S3 Lifecycle configuration\. Each example shows how you can specify the XML in each of the example scenarios\.

**Topics**
+ [Example 1: Specifying a filter](#lifecycle-config-ex1)
+ [Example 2: Disabling a Lifecycle rule](#lifecycle-config-conceptual-ex2)
+ [Example 3: Tiering down storage class over an object's lifetime](#lifecycle-config-conceptual-ex3)
+ [Example 4: Specifying multiple rules](#lifecycle-config-conceptual-ex4)
+ [Example 5: Overlapping filters, conflicting lifecycle actions, and what Amazon S3 does with nonversioned buckets](#lifecycle-config-conceptual-ex5)
+ [Example 6: Specifying a lifecycle rule for a versioning\-enabled bucket](#lifecycle-config-conceptual-ex6)
+ [Example 7: Removing expired object delete markers](#lifecycle-config-conceptual-ex7)
+ [Example 8: Lifecycle configuration to abort multipart uploads](#lc-expire-mpu)
+ [Example 9: Lifecycle configuration using size\-based rules](#lc-size-rules)

## Example 1: Specifying a filter<a name="lifecycle-config-ex1"></a>

Each S3 Lifecycle rule includes a filter that you can use to identify a subset of objects in your bucket to which the S3 Lifecycle rule applies\. The following S3 Lifecycle configurations show examples of how you can specify a filter\.
+ In this S3 Lifecycle configuration rule, the filter specifies a key prefix \(`tax/`\)\. Therefore, the rule applies to objects with the key name prefix `tax/`, such as `tax/doc1.txt` and `tax/doc2.txt`\.

  The rule specifies two actions that direct Amazon S3 to do the following:
  + Transition objects to the S3 Glacier Flexible Retrieval storage class 365 days \(one year\) after creation\.
  + Delete objects \(the `Expiration` action\) 3,650 days \(10 years\) after creation\.

  ```
  <LifecycleConfiguration>
    <Rule>
      <ID>Transition and Expiration Rule</ID>
      <Filter>
         <Prefix>tax/</Prefix>
      </Filter>
      <Status>Enabled</Status>
      <Transition>
        <Days>365</Days>
        <StorageClass>S3 Glacier Flexible Retrieval</StorageClass>
      </Transition>
      <Expiration>
        <Days>3650</Days>
      </Expiration>
    </Rule>
  </LifecycleConfiguration>
  ```

  Instead of specifying object age in terms of days after creation, you can specify a date for each action\. However, you can't use both `Date` and `Days` in the same rule\. 
+ If you want the S3 Lifecycle rule to apply to all objects in the bucket, specify an empty prefix\. In the following configuration, the rule specifies a `Transition` action that directs Amazon S3 to transition objects to the S3 Glacier Flexible Retrieval storage class 0 days after creation\. This rule means that the objects are eligible for archival to Amazon S3 Glacier at midnight UTC following creation\. For more information about lifecycle constraints, see [Constraints](lifecycle-transition-general-considerations.md#lifecycle-configuration-constraints)\.

  ```
  <LifecycleConfiguration>
    <Rule>
      <ID>Archive all object same-day upon creation</ID>
      <Filter>
        <Prefix></Prefix>
      </Filter>
      <Status>Enabled</Status>
      <Transition>
        <Days>0</Days>
        <StorageClass>S3 Glacier Flexible Retrieval</StorageClass>
      </Transition>
    </Rule>
  </LifecycleConfiguration>
  ```
+ You can specify zero or one key name prefix and zero or more object tags in a filter\. The following example code applies the S3 Lifecycle rule to a subset of objects with the `tax/` key prefix and to objects that have two tags with specific key and value\. When you specify more than one filter, you must include the `<And>` element as shown \(Amazon S3 applies a logical `AND` to combine the specified filter conditions\)\.

  ```
  ...
  <Filter>
     <And>
        <Prefix>tax/</Prefix>
        <Tag>
           <Key>key1</Key>
           <Value>value1</Value>
        </Tag>
        <Tag>
           <Key>key2</Key>
           <Value>value2</Value>
        </Tag>
      </And>
  </Filter>
  ...
  ```

  
+ You can filter objects based only on tags\. For example, the following S3 Lifecycle rule applies to objects that have the two specified tags \(it does not specify any prefix\)\.

  ```
  ...
  <Filter>
     <And>
        <Tag>
           <Key>key1</Key>
           <Value>value1</Value>
        </Tag>
        <Tag>
           <Key>key2</Key>
           <Value>value2</Value>
        </Tag>
      </And>
  </Filter>
  ...
  ```

  

**Important**  
When you have multiple rules in an S3 Lifecycle configuration, an object can become eligible for multiple S3 Lifecycle actions\. In such cases, Amazon S3 follows these general rules:  
Permanent deletion takes precedence over transition\.
Transition takes precedence over creation of delete markers\.
When an object is eligible for both a S3 Glacier Flexible Retrieval and S3 Standard\-IA \(or S3 One Zone\-IA\) transition, Amazon S3 chooses the S3 Glacier Flexible Retrieval transition\.
 For examples, see [Example 5: Overlapping filters, conflicting lifecycle actions, and what Amazon S3 does with nonversioned buckets](#lifecycle-config-conceptual-ex5)\. 



## Example 2: Disabling a Lifecycle rule<a name="lifecycle-config-conceptual-ex2"></a>

You can temporarily disable a S3 Lifecycle rule\. The following S3 Lifecycle configuration specifies two rules:
+ Rule 1 directs Amazon S3 to transition objects with the `logs/` prefix to the S3 Glacier Flexible Retrieval storage class soon after creation\. 
+ Rule 2 directs Amazon S3 to transition objects with the `documents/` prefix to the S3 Glacier Flexible Retrieval storage class soon after creation\. 

In the policy, Rule 1 is enabled and Rule 2 is disabled\. Amazon S3 ignores disabled rules\.

```
<LifecycleConfiguration>
  <Rule>
    <ID>Rule1</ID>
    <Filter>
      <Prefix>logs/</Prefix>
    </Filter>
    <Status>Enabled</Status>
    <Transition>
      <Days>0</Days>
      <StorageClass>S3 Glacier Flexible Retrieval</StorageClass>
    </Transition>
  </Rule>
  <Rule>
    <ID>Rule2</ID>
    <Prefix>documents/</Prefix>
    <Status>Disabled</Status>
    <Transition>
      <Days>0</Days>
      <StorageClass>S3 Glacier Flexible Retrieval</StorageClass>
    </Transition>
  </Rule>
</LifecycleConfiguration>
```

## Example 3: Tiering down storage class over an object's lifetime<a name="lifecycle-config-conceptual-ex3"></a>

In this example, you use S3 Lifecycle configuration to tier down the storage class of objects over their lifetime\. Tiering down can help reduce storage costs\. For more information about pricing, see [Amazon S3 pricing](https://aws.amazon.com/s3/pricing/)\.

The following S3 Lifecycle configuration specifies a rule that applies to objects with the key name prefix `logs/`\. The rule specifies the following actions:
+ Two transition actions:
  + Transition objects to the S3 Standard\-IA storage class 30 days after creation\.
  + Transition objects to the S3 Glacier Flexible Retrieval storage class 90 days after creation\.
+ One expiration action that directs Amazon S3 to delete objects a year after creation\.

```
<LifecycleConfiguration>
  <Rule>
    <ID>example-id</ID>
    <Filter>
       <Prefix>logs/</Prefix>
    </Filter>
    <Status>Enabled</Status>
    <Transition>
      <Days>30</Days>
      <StorageClass>STANDARD_IA</StorageClass>
    </Transition>
    <Transition>
      <Days>90</Days>
      <StorageClass>GLACIER</StorageClass>
    </Transition>
    <Expiration>
      <Days>365</Days>
    </Expiration>
  </Rule>
</LifecycleConfiguration>
```

**Note**  
You can use one rule to describe all S3 Lifecycle actions if all actions apply to the same set of objects \(identified by the filter\)\. Otherwise, you can add multiple rules with each specifying a different filter\.

## Example 4: Specifying multiple rules<a name="lifecycle-config-conceptual-ex4"></a>



You can specify multiple rules if you want different S3 Lifecycle actions of different objects\. The following S3 Lifecycle configuration has two rules:
+ Rule 1 applies to objects with the key name prefix `classA/`\. It directs Amazon S3 to transition objects to the S3 Glacier Flexible Retrieval storage class one year after creation and expire these objects 10 years after creation\.
+ Rule 2 applies to objects with key name prefix `classB/`\. It directs Amazon S3 to transition objects to the S3 Standard\-IA storage class 90 days after creation and delete them one year after creation\.

```
<LifecycleConfiguration>
    <Rule>
        <ID>ClassADocRule</ID>
        <Filter>
           <Prefix>classA/</Prefix>        
        </Filter>
        <Status>Enabled</Status>
        <Transition>        
           <Days>365</Days>        
           <StorageClass>GLACIER</StorageClass>       
        </Transition>    
        <Expiration>
             <Days>3650</Days>
        </Expiration>
    </Rule>
    <Rule>
        <ID>ClassBDocRule</ID>
        <Filter>
            <Prefix>classB/</Prefix>
        </Filter>
        <Status>Enabled</Status>
        <Transition>        
           <Days>90</Days>        
           <StorageClass>STANDARD_IA</StorageClass>       
        </Transition>    
        <Expiration>
             <Days>365</Days>
        </Expiration>
    </Rule>
</LifecycleConfiguration>
```

## Example 5: Overlapping filters, conflicting lifecycle actions, and what Amazon S3 does with nonversioned buckets<a name="lifecycle-config-conceptual-ex5"></a>

You might specify an S3 Lifecycle configuration in which you specify overlapping prefixes, or actions\.

Generally, S3 Lifecycle optimizes for cost\. For example, if two expiration policies overlap, the shorter expiration policy is honored so that data is not stored for longer than expected\. Likewise, if two transition policies overlap, S3 Lifecycle transitions your objects to the lower\-cost storage class\. 

In both cases, S3 Lifecycle tries to choose the path that is least expensive for you\. An exception to this general rule is with the S3 Intelligent\-Tiering storage class\. S3 Intelligent\-Tiering is favored by S3 Lifecycle over any storage class, aside from the S3 Glacier Flexible Retrieval and S3 Glacier Deep Archive storage classes\.

The following examples show how Amazon S3 resolves potential conflicts\.

**Example 1: Overlapping prefixes \(no conflict\)**  
The following example configuration has two rules that specify overlapping prefixes as follows:  
+ The first rule specifies an empty filter, indicating all objects in the bucket\. 
+ The second rule specifies a key name prefix \(`logs/`\), indicating only a subset of objects\.
Rule 1 requests Amazon S3 to delete all objects one year after creation\. Rule 2 requests Amazon S3 to transition a subset of objects to the S3 Standard\-IA storage class 30 days after creation\.  

```
 1. <LifecycleConfiguration>
 2.   <Rule>
 3.     <ID>Rule 1</ID>
 4.     <Filter>
 5.     </Filter>
 6.     <Status>Enabled</Status>
 7.     <Expiration>
 8.       <Days>365</Days>
 9.     </Expiration>
10.   </Rule>
11.   <Rule>
12.     <ID>Rule 2</ID>
13.     <Filter>
14.       <Prefix>logs/</Prefix>
15.     </Filter>
16.     <Status>Enabled</Status>
17.     <Transition>
18.       <StorageClass>STANDARD_IA<StorageClass>
19.       <Days>30</Days>
20.     </Transition>
21.    </Rule>
22. </LifecycleConfiguration>
```
Since there is no conflict in this case, Amazon S3 will transition the objects with the `logs/` prefix to the S3 Standard\-IA storage class 30 days after creation\. When any object reaches one year after creation, it will be deleted\.

**Example 2: Conflicting lifecycle actions**  
In this example configuration, there are two rules that direct Amazon S3 to perform two different actions on the same set of objects at the same time in the objects' lifetime:  
+ Both rules specify the same key name prefix, so both rules apply to the same set of objects\.
+ Both rules specify the same 365 days after object creation when the rules apply\.
+ One rule directs Amazon S3 to transition objects to the S3 Standard\-IA storage class and another rule wants Amazon S3 to expire the objects at the same time\.

```
<LifecycleConfiguration>
  <Rule>
    <ID>Rule 1</ID>
    <Filter>
      <Prefix>logs/</Prefix>
    </Filter>
    <Status>Enabled</Status>
    <Expiration>
      <Days>365</Days>
    </Expiration>        
  </Rule>
  <Rule>
    <ID>Rule 2</ID>
    <Filter>
      <Prefix>logs/</Prefix>
    </Filter>
    <Status>Enabled</Status>
    <Transition>
      <StorageClass>STANDARD_IA<StorageClass>
      <Days>365</Days>
    </Transition>
   </Rule>
</LifecycleConfiguration>
```
In this case, because you want objects to expire \(to be removed\), there is no point in changing the storage class, so Amazon S3 chooses the expiration action on these objects\.

**Example 3: Overlapping prefixes resulting in conflicting lifecycle actions**  
In this example, the configuration has two rules, which specify overlapping prefixes as follows:  
+ Rule 1 specifies an empty prefix \(indicating all objects\)\.
+ Rule 2 specifies a key name prefix \(`logs/`\) that identifies a subset of all objects\.
For the subset of objects with the `logs/` key name prefix, S3 Lifecycle actions in both rules apply\. One rule directs Amazon S3 to transition objects 10 days after creation, and another rule directs Amazon S3 to transition objects 365 days after creation\.   

```
<LifecycleConfiguration>
  <Rule>
    <ID>Rule 1</ID>
    <Filter>
      <Prefix></Prefix>
    </Filter>
    <Status>Enabled</Status>
    <Transition>
      <StorageClass>STANDARD_IA<StorageClass>
      <Days>10</Days> 
    </Transition>
  </Rule>
  <Rule>
    <ID>Rule 2</ID>
    <Filter>
      <Prefix>logs/</Prefix>
    </Filter>
    <Status>Enabled</Status>
    <Transition>
      <StorageClass>STANDARD_IA<StorageClass>
      <Days>365</Days> 
    </Transition>
   </Rule>
</LifecycleConfiguration>
```
In this case, Amazon S3 chooses to transition them 10 days after creation\. 

**Example 4: Tag\-based filtering and resulting conflicting lifecycle actions**  
Suppose that you have the following S3 Lifecycle policy that has two rules, each specifying a tag filter:  
+ Rule 1 specifies a tag\-based filter \(`tag1/value1`\)\. This rule directs Amazon S3 to transition objects to the S3 Glacier Flexible Retrieval storage class 365 days after creation\.
+ Rule 2 specifies a tag\-based filter \(`tag2/value2`\)\. This rule directs Amazon S3 to expire objects 14 days after creation\.
The S3 Lifecycle configuration is shown in following example\.  

```
<LifecycleConfiguration>
  <Rule>
    <ID>Rule 1</ID>
    <Filter>
      <Tag>
         <Key>tag1</Key>
         <Value>value1</Value>
      </Tag>
    </Filter>
    <Status>Enabled</Status>
    <Transition>
      <StorageClass>GLACIER<StorageClass>
      <Days>365</Days> 
    </Transition>
  </Rule>
  <Rule>
    <ID>Rule 2</ID>
    <Filter>
      <Tag>
         <Key>tag2</Key>
         <Value>value2</Value>
      </Tag>
    </Filter>
    <Status>Enabled</Status>
    <Expiration>
      <Days>14</Days> 
    </Expiration>
   </Rule>
</LifecycleConfiguration>
```
If an object has both tags, then Amazon S3 has to decide which rule to follow\. In this case, Amazon S3 expires the object 14 days after creation\. The object is removed, and therefore the transition action does not apply\.





## Example 6: Specifying a lifecycle rule for a versioning\-enabled bucket<a name="lifecycle-config-conceptual-ex6"></a>

Suppose that you have a versioning\-enabled bucket, which means that for each object, you have a current version and zero or more noncurrent versions\. \(For more information about S3 Versioning, see [Using versioning in S3 buckets](Versioning.md)\.\) In this example, you want to maintain one year's worth of history, and delete the noncurrent versions\. S3 Lifecycle configurations supports keeping 1 to 100 versions of any object\. 

To save storage costs, you want to move noncurrent versions to S3 Glacier Flexible Retrieval 30 days after they become noncurrent \(assuming that these noncurrent objects are cold data for which you don't need real\-time access\)\. In addition, you expect frequency of access of the current versions to diminish 90 days after creation, so you might choose to move these objects to the S3 Standard\-IA storage class\.

```
 1. <LifecycleConfiguration>
 2.     <Rule>
 3.         <ID>sample-rule</ID>
 4.         <Filter>
 5.            <Prefix></Prefix>
 6.         </Filter>
 7.         <Status>Enabled</Status>
 8.         <Transition>
 9.            <Days>90</Days>
10.            <StorageClass>STANDARD_IA</StorageClass>
11.         </Transition>
12.         <NoncurrentVersionTransition>      
13.             <NoncurrentDays>30</NoncurrentDays>      
14.             <StorageClass>S3 Glacier Flexible Retrieval</StorageClass>   
15.         </NoncurrentVersionTransition>    
16.        <NoncurrentVersionExpiration>     
17.             <NewerNoncurrentVersions>5</NewerNoncurrentVersions>
18.             <NoncurrentDays>365</NoncurrentDays>    
19.        </NoncurrentVersionExpiration> 
20.     </Rule>
21. </LifecycleConfiguration>
```

## Example 7: Removing expired object delete markers<a name="lifecycle-config-conceptual-ex7"></a>



A versioning\-enabled bucket has one current version and zero or more noncurrent versions for each object\. When you delete an object, note the following:
+ If you don't specify a version ID in your delete request, Amazon S3 adds a delete marker instead of deleting the object\. The current object version becomes noncurrent, and the delete marker becomes the current version\. 
+ If you specify a version ID in your delete request, Amazon S3 deletes the object version permanently \(a delete marker is not created\)\.
+ A delete marker with zero noncurrent versions is referred to as an *expired object delete marker*\. 

This example shows a scenario that can create expired object delete markers in your bucket, and how you can use S3 Lifecycle configuration to direct Amazon S3 to remove the expired object delete markers\.

Suppose that you write a S3 Lifecycle policy that uses the `NoncurrentVersionExpiration` action to remove the noncurrent versions 30 days after they become noncurrent and retains at most 10 noncurrent versions, as shown in the following example\.

```
<LifecycleConfiguration>
    <Rule>
        ...
        <NoncurrentVersionExpiration>     
            <NewerNoncurrentVersions>10</NewerNoncurrentVersions>
            <NoncurrentDays>30</NoncurrentDays>    
        </NoncurrentVersionExpiration>
    </Rule>
</LifecycleConfiguration>
```

The `NoncurrentVersionExpiration` action does not apply to the current object versions\. It removes only the noncurrent versions\.

For current object versions, you have the following options to manage their lifetime, depending on whether the current object versions follow a well\-defined lifecycle: 
+ **The current object versions follow a well\-defined lifecycle\.**

  In this case, you can use an S3 Lifecycle policy with the `Expiration` action to direct Amazon S3 to remove the current versions, as shown in the following example\.

  ```
  <LifecycleConfiguration>
      <Rule>
          ...
          <Expiration>
             <Days>60</Days>
          </Expiration>
          <NoncurrentVersionExpiration>     
              <NewerNoncurrentVersions>10</NewerNoncurrentVersions>
              <NoncurrentDays>30</NoncurrentDays>    
          </NoncurrentVersionExpiration>
      </Rule>
  </LifecycleConfiguration>
  ```

  In this example, Amazon S3 removes current versions 60 days after they are created by adding a delete marker for each of the current object versions\. This process makes the current version noncurrent, and the delete marker becomes the current version\. For more information, see [Using versioning in S3 buckets](Versioning.md)\. 
**Note**  
You cannot specify both a `Days` and an `ExpiredObjectDeleteMarker` tag on the same rule\. When you specify the `Days` tag, Amazon S3 automatically performs `ExpiredObjectDeleteMarker` cleanup when the delete markers are old enough to satisfy the age criteria\. To clean up delete markers as soon as they become the only version, create a separate rule with only the `ExpiredObjectDeleteMarker` tag\.

  The `NoncurrentVersionExpiration` action in the same S3 Lifecycle configuration removes noncurrent objects 30 days after they become noncurrent\. Thus, in this example, all object versions are permanently removed 90 days after object creation\. Although expired object delete markers are created during this process, Amazon S3 detects and removes the expired object delete markers for you\. 
+ **The current object versions don't have a well\-defined lifecycle\.** 

  In this case, you might remove the objects manually when you don't need them, creating a delete marker with one or more noncurrent versions\. If your S3 Lifecycle configuration with the `NoncurrentVersionExpiration` action removes all the noncurrent versions, you now have expired object delete markers\.

  Specifically for this scenario, S3 Lifecycle configuration provides an `Expiration` action that you can use to remove the expired object delete markers\.

  

  ```
  <LifecycleConfiguration>
      <Rule>
         <ID>Rule 1</ID>
          <Filter>
            <Prefix>logs/</Prefix>
          </Filter>
          <Status>Enabled</Status>
          <Expiration>
             <ExpiredObjectDeleteMarker>true</ExpiredObjectDeleteMarker>
          </Expiration>
          <NoncurrentVersionExpiration>     
              <NewerNoncurrentVersions>10</NewerNoncurrentVersions>
              <NoncurrentDays>30</NoncurrentDays>    
          </NoncurrentVersionExpiration>
      </Rule>
  </LifecycleConfiguration>
  ```

By setting the `ExpiredObjectDeleteMarker` element to `true` in the `Expiration` action, you direct Amazon S3 to remove the expired object delete markers\.

**Note**  
When you use the `ExpiredObjectDeleteMarker` S3 Lifecycle action, the rule cannot specify a tag\-based filter\.

## Example 8: Lifecycle configuration to abort multipart uploads<a name="lc-expire-mpu"></a>

You can use the Amazon S3 multipart upload REST API operations to upload large objects in parts\. For more information about multipart uploads, see [Uploading and copying objects using multipart upload](mpuoverview.md)\. 

Using S3 Lifecycle configuration, you can direct Amazon S3 to stop incomplete multipart uploads \(identified by the key name prefix specified in the rule\) if they aren't completed within a specified number of days after initiation\. When Amazon S3 aborts a multipart upload, it deletes all the parts associated with the multipart upload\. This process helps control your storage costs by ensuring that you don't have incomplete multipart uploads with parts that are stored in Amazon S3\. 

**Note**  
When you use the `AbortIncompleteMultipartUpload` S3 Lifecycle action, the rule cannot specify a tag\-based filter\.

The following is an example S3 Lifecycle configuration that specifies a rule with the `AbortIncompleteMultipartUpload` action\. This action directs Amazon S3 to stop incomplete multipart uploads seven days after initiation\.

```
<LifecycleConfiguration>
    <Rule>
        <ID>sample-rule</ID>
        <Filter>
           <Prefix>SomeKeyPrefix/</Prefix>
        </Filter>
        <Status>rule-status</Status>
        <AbortIncompleteMultipartUpload>
          <DaysAfterInitiation>7</DaysAfterInitiation>
        </AbortIncompleteMultipartUpload>
    </Rule>
</LifecycleConfiguration>
```

## Example 9: Lifecycle configuration using size\-based rules<a name="lc-size-rules"></a>

You can create rules that transition objects based only on their size\. You can specify a minimum size \(`ObjectSizeGreaterThan`\) or a maximum size \(`ObjectSizeLessThan`\), or you can specify a range of object sizes\. When using more than one filter, such as a prefix and size rule, you must wrap the filters in an `<And>` element\.

```
<LifecycleConfiguration>
  <Rule>
    <ID>Transition with a prefix and based on size</ID>
    <Filter>
       <And>
          <Prefix>tax/</Prefix>
          <ObjectSizeGreaterThan>500</ObjectSizeGreaterThan>
       </And>   
    </Filter>
    <Status>Enabled</Status>
    <Transition>
      <Days>365</Days>
      <StorageClass>S3 Glacier Flexible Retrieval</StorageClass>
    </Transition>
  </Rule>
</LifecycleConfiguration>
```

If you're specifying a range by using both the `ObjectSizeGreaterThan` and `ObjectSizeLessThan` elements, the maximum object size must be larger than the minimum object size\. When using more than one filter, you must wrap the filters in an `<And>` element\. The following example shows how to specify objects in a range between 500 and 64000 bytes\. 

```
<LifecycleConfiguration>
    <Rule>
        ...
          <And>
             <ObjectSizeGreaterThan>500</ObjectSizeGreaterThan>
             <ObjectSizeLessThan>64000</ObjectSizeLessThan>
          </And>
    </Rule>
</LifecycleConfiguration>
```