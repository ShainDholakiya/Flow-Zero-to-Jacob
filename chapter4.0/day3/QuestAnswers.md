1.

- Cannot store multiple NFTs easily
- No one can mint an NFT for someone else

2.  Create a destroy function for the field that has another resource as its type

```
destroy () {
  destroy self.xxx
}
```

3.

- We could restrict access to `createNFT()` by having it just be the account owner. We could also restrict it to a specific set of addresses like a whitelist.
- We could create another function inside the colllection resource that allows people to borrow the NFT. This way we would just return back a reference of the NFT they want to access from the `ownedNFTs` array instead of them having to actually borrow from the collection itself.
- We could also add a MetadataView
