1. An event is simply a way to show the world that something just happened. You would first define an event and then call emit on that event when you want to show it on the blockchain. This is useful for the client side of an application because it would allow you to be able to show the user confirmation of events that are happening in real-time much easier.
2.

```
pub contract CryptoPoops {
  pub event NFTMinted(id: UInt64)
  pub event NFTWithdrawn(id: UInt64)
  pub event NFTDeposited(id: UInt64, name: String, favouriteFood: String, luckyNumber: Int)

  pub var totalSupply: UInt64

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

      CryptoPoops.totalSupply = CryptoPoops.totalSupply + 1

      emit NFTMinted(id: self.id)
    }
  }

  pub resource interface CollectionPublic {
    pub fun deposit(token: @NFT)
    pub fun getIDs(): [UInt64]
    pub fun borrowNFT(id: UInt64): &NFT
  }

  pub resource Collection: CollectionPublic {
    pub var ownedNFTs: @{UInt64: NFT}

    pub fun deposit(token: @NFT) {
      let id = token.id
      self.ownedNFTs[token.id] <-! token
      let nft = &self.ownedNFTs[id] as &NFT
      emit NFTDeposited(id: nft.id, name: nft.name, favouriteFood: nft.favouriteFood, luckyNumber: nft.luckyNumber)
    }

    pub fun withdraw(withdrawID: UInt64): @NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID)
              ?? panic("This NFT does not exist in this Collection.")
      emit NFTWithdrawn(id: withdrawID)
      return <- nft
    }

    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
    }

    pub fun borrowNFT(id: UInt64): &NFT {
      return &self.ownedNFTs[id] as &NFT
    }

    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }

  pub fun createEmptyCollection(): @Collection {
    return <- create Collection()
  }

  pub resource Minter {

    pub fun createNFT(name: String, favouriteFood: String, luckyNumber: Int): @NFT {
      return <- create NFT(_name: name, _favouriteFood: favouriteFood, _luckyNumber: luckyNumber)
    }

    pub fun createMinter(): @Minter {
      return <- create Minter()
    }

  }

  init() {
    self.totalSupply = 0

    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
```

3.

```
pub contract CryptoPoops {
  pub event NFTMinted(id: UInt64)
  pub event NFTWithdrawn(id: UInt64)
  pub event NFTDeposited(id: UInt64, name: String, favouriteFood: String, luckyNumber: Int)

  pub var totalSupply: UInt64

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

      CryptoPoops.totalSupply = CryptoPoops.totalSupply + 1

      emit NFTMinted(id: self.id)
    }
  }

  pub resource interface CollectionPublic {
    pub fun deposit(token: @NFT)
    pub fun getIDs(): [UInt64]
    pub fun borrowNFT(id: UInt64): &NFT
  }

  pub resource Collection: CollectionPublic {
    pub var ownedNFTs: @{UInt64: NFT}

    pub fun deposit(token: @NFT) {
      let id = token.id
      self.ownedNFTs[token.id] <-! token
      let nft = &self.ownedNFTs[id] as &NFT
      emit NFTDeposited(id: nft.id, name: nft.name, favouriteFood: nft.favouriteFood, luckyNumber: nft.luckyNumber)
    }

    pub fun withdraw(withdrawID: UInt64): @NFT {
      post {
        self.ownedNFTs.length == before(self.ownedNFTs.length) - 1
      }
      let nft <- self.ownedNFTs.remove(key: withdrawID)
              ?? panic("This NFT does not exist in this Collection.")
      emit NFTWithdrawn(id: withdrawID)
      return <- nft
    }

    pub fun getIDs(): [UInt64] {
      pre {
        self.ownedNFTs.length != 0: "There are no NFTs in this Collection dude. Run me again where there are some to see the IDs."
      }
      return self.ownedNFTs.keys
    }

    pub fun borrowNFT(id: UInt64): &NFT {
      pre {
        self.ownedNFTs[id] != nil: "ID does not exist in this Collection"
      }
      return &self.ownedNFTs[id] as &NFT
    }

    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }

  pub fun createEmptyCollection(): @Collection {
    return <- create Collection()
  }

  pub resource Minter {

    pub fun createNFT(name: String, favouriteFood: String, luckyNumber: Int): @NFT {
      post {
        CryptoPoops.totalSupply == before(CryptoPoops.totalSupply) + 1
      }
      return <- create NFT(_name: name, _favouriteFood: favouriteFood, _luckyNumber: luckyNumber)
    }

    pub fun createMinter(): @Minter {
      return <- create Minter()
    }

  }

  init() {
    self.totalSupply = 0

    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
```

4.

- numberOne(): It will log the name becausse 'Jacob' has a length of 5
- numberTwo(): It will return a value
- numberThree(): It will not log the updated number since the post will not pass. `number` will be 1 and the post checks if the previous state of `number` (0) is equal to `result + 1` (2).
