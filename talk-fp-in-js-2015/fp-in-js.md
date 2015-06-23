FP in JS Challenges
===================

### Pros

Instance method -> function

```javascript
var describeEnvironmentResources = denodeify(elasticbeanstalk.describeEnvironmentResources).bind(elasticbeanstalk);
```

### Working with trees

Lists are easy but what about
```javascript
var data = [{'Environment123': ['Instance1', 'Instance2']}];
```
Task: Map `getInstanceIp` over instance IDs.

### Object cannot be iterated easily

Here I'd like to apply `filter` but cannot because this is an object
(workaround: `pairs` ... `zip`):

```javascript
// Remove those contentful members for which there is no corresponding hardwareCatalog entry
    _.forEach(contentfulHardwareFamily.members, function(familyMember, guid) {
        var foundInHardwareCatalog = _.isObject(hardwareCatalog[guid]);
        if (!foundInHardwareCatalog) {
            delete contentfulHardwareFamily.members[guid];
            logger.warn("A Contentful GUID (product) exists which is not in the HardwareCatalog:", contentfulHardwareFamily.name, guid);
        }
    });
```

### Immutability would require (deep)cloning all the time => forEach more practical

```javascript
_.forEach(hardwareFamiliesContentful, function(contentfulHardwareFamily) {

      // Get the full accessories (linked via contentful) from the hardwareCatalog
      var accessoriesForThisHardwareFamily = [];
      _.forEach(contentfulHardwareFamily.accessories, function(contentfulAccessory) {
          if (_.isEmpty(contentfulAccessory)) { return; } // Unpublished contentful accessories are sometimes returned as undefined

          var foundInHardwareCatalog = _.isObject(hardwareCatalog[contentfulAccessory.guid]);
          if (!foundInHardwareCatalog) {
              logger.warn("A Contentful GUID (accessory) exists which is not in the HardwareCatalog:", contentfulAccessory.name, contentfulAccessory.guid);
          } else {
              accessoriesForThisHardwareFamily.push( hardwareCatalog[contentfulAccessory.guid] );
          }
      });
})
```

## Icepick

### Pros

* updateIn, assocIn, ...

### Cons

* Combine with `lodash`? Less functionality than it
* No "`chain`" (yet)

## Lodash vs. Clojure[Script]

* No immutable data support
* No `partition-with`
