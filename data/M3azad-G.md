### High gas cost using storage array itteration 

```
 function _addOwnedCurvesTokenSubject(address owner_, address curvesTokenSubject) internal {
        address[] storage subjects = ownedCurvesTokenSubjects[owner_];  //@audit huge gas cost for itterating over storage variables..
        for (uint256 i = 0; i < subjects.length; i++) {
            if (subjects[i] == curvesTokenSubject) {
                return;
            }
        }
        subjects.push(curvesTokenSubject);
    }
```

In the above function user of arrays to store owned subject address , not only array , `storage` array is used to get Subject . and itteration of storage array can cause a lot of gas. use of `memory` is recommed insted of storage . 

Not only this, but a mapping or any other alteranative should be used for storing subjects.