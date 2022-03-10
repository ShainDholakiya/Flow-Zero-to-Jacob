1. Structs can be copied, can be lost or overwritten, and created whenever you want.
2. It would be a better decsion to use a resource instead of a struct when handling owners of some important data so the owner can't be copied.
3. `create`
4. No, you can only create a resource in the same contract it was defined in.
5. `@Jacob`
6. 
```
pub fun createJacob(): @Jacob {
    let myJacob <- create Jacob()
    return <- myJacob
}
```
