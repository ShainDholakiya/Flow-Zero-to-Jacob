```
pub contract CryptoPoops {
  pub var totalSupply: UInt64

  // This is an NFT resource that contains an id, name, favouriteFood, and luckyNumber
  pub resource NFT {
    pub let id: UInt64

    pub let name: String
    pub let favouriteFood: String
    pub let luckyNumber: Int

    init(_name: String, _favouriteFood: String, _luckyNumber: Int) {
      self.id = self.uuid

      self.name = _name
      self.favouriteFood = _favouriteFood
      self.luckyNumber = _luckyNumber
    }
  }

  // This is a resource interface consisting of a deposit, getIDs, and borrowNFT functions
  pub resource interface CollectionPublic {
    pub fun deposit(token: @NFT)
    pub fun getIDs(): [UInt64]
    pub fun borrowNFT(id: UInt64): &NFT
  }

  // This is a Collection resource that implements the CollectionPublic resource interface which means it must include the three functions listed in CollectionPublic
  pub resource Collection: CollectionPublic {
    pub var ownedNFTs: @{UInt64: NFT}

    // This function adds the NFT token to the ownedNFTs dictionary and will only be called by the account owner of this contract since they will have the Minter in /storage/Minter to actually be able to create an NFT and then deposit it into someone else's Collection
    // Since NFT is a resource, the syntax includes a <- and we use ! to force it
    pub fun deposit(token: @NFT) {
      self.ownedNFTs[token.id] <-! token
    }

    // This function removes an NFT from the ownedNFTs dictionary and will only be able to be called if the user has a Collectin in their /storage/Collection
    pub fun withdraw(withdrawID: UInt64): @NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID)
              ?? panic("This NFT does not exist in this Collection.")
      return <- nft
    }

    // This function simply returns all the keys of the signer's ownedNFTs from their Collection
    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
    }

    // This function allows us to read an NFT from a Collection without having to withdraw the NFT from the Collection itself
    pub fun borrowNFT(id: UInt64): &NFT {
      return &self.ownedNFTs[id] as &NFT
    }

    init() {
      self.ownedNFTs <- {}
    }

    // This function is needed for nested resources
    destroy() {
      destroy self.ownedNFTs
    }
  }

  // This function creates an empty Collection which we will then use to store at /storage/Collection
  // The benefit of using a Collection is so that you can store multiple things in one place and have easy access to them rather than storing at /storage/NFT1, /storage/NFT2, etc.
  pub fun createEmptyCollection(): @Collection {
    return <- create Collection()
  }

  // This is a Minter resource that consists of a createNFT and createMinter function
  pub resource Minter {

    // This function creates an NFT
    pub fun createNFT(name: String, favouriteFood: String, luckyNumber: Int): @NFT {
      return <- create NFT(_name: name, _favouriteFood: favouriteFood, _luckyNumber: luckyNumber)
    }

    // This function creates a Minter
    pub fun createMinter(): @Minter {
      return <- create Minter()
    }

  }

  init() {
    self.totalSupply = 0

    // This will create and save a Minter to the account owner of this contract at /storage/Minter
    // This allows only the account owner of this contract to be able to call createNFT()
    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
```
