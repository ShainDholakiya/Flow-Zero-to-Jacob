1. "force casting" allows us to "downcast" a type into a more specific type. In our case, it helped us so we could "downcast" the `NonFungibleToken.NFT` into our own `NFT` reference so we could read oether attributes that are custom to ours.
2. In order to "downcast", you need an "authorized reference" which is marked by the `auth` keyword.
3.

```
https://github.com/onflow/flow-nft/blob/master/contracts/NonFungibleToken.cdc
```

```
import NonFungibleToken from 0x01

pub contract CryptoPoops: NonFungibleToken {
  pub var totalSupply: UInt64

  pub event ContractInitialized()
  pub event Withdraw(id: UInt64, from: Address?)
  pub event Deposit(id: UInt64, to: Address?)

  pub resource NFT: NonFungibleToken.INFT {
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

  pub resource interface CustomCollectionPublic {
    pub fun borrowAuthNFT(id: UInt64): &NFT
  }

  pub resource Collection: NonFungibleToken.Provider, NonFungibleToken.Receiver, NonFungibleToken.CollectionPublic, CustomCollectionPublic {
    pub var ownedNFTs: @{UInt64: NonFungibleToken.NFT}

    pub fun withdraw(withdrawID: UInt64): @NonFungibleToken.NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID)
            ?? panic("This NFT does not exist in this Collection.")
      emit Withdraw(id: nft.id, from: self.owner?.address)
      return <- nft
    }

    pub fun deposit(token: @NonFungibleToken.NFT) {
      let nft <- token as! @NFT
      emit Deposit(id: nft.id, to: self.owner?.address)
      self.ownedNFTs[nft.id] <-! nft
    }

    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
    }

    pub fun borrowNFT(id: UInt64): &NonFungibleToken.NFT {
      return &self.ownedNFTs[id] as &NonFungibleToken.NFT
    }

    pub fun borrowAuthNFT(id: UInt64): &NFT {
      let ref = &self.ownedNFTs[id] as auth &NonFungibleToken.NFT
      return ref as! &NFT
    }

    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }

  pub fun createEmptyCollection(): @NonFungibleToken.Collection {
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
    emit ContractInitialized()
    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
```

```
import NonFungibleToken from 0x01
import CryptoPoops from 0x02

transaction() {

  prepare(acct: AuthAccount) {
    acct.save(<- CryptoPoops.createEmptyCollection(), to: /storage/Collection)
    acct.link<&CryptoPoops.Collection{NonFungibleToken.CollectionPublic, CryptoPoops.CustomCollectionPublic}>(/public/Collection, target: /storage/Collection)
  }

  execute {
    log("Collection created")
  }
}
```

```
import NonFungibleToken from 0x01
import CryptoPoops from 0x02

transaction(recipient: Address, name: String, favouriteFood: String, luckyNumber: Int) {

  prepare(acct: AuthAccount) {
    let nftMinter = acct.borrow<&CryptoPoops.Minter>(from: /storage/Minter)!

    let pubRef = getAccount(recipient).getCapability(/public/Collection)
                  .borrow<&CryptoPoops.Collection{NonFungibleToken.CollectionPublic}>()
                  ?? panic("This account does not have a Collection")

    pubRef.deposit(token: <- nftMinter.createNFT(name: name, favouriteFood: favouriteFood, luckyNumber: luckyNumber))
  }

  execute {
    log("NFT minted")
  }
}
```

```
import NonFungibleToken from 0x01
import CryptoPoops from 0x02

pub fun main(account: Address, id: UInt64): String {
    let pubRef = getAccount(account).getCapability(/public/Collection)
                  .borrow<&CryptoPoops.Collection{NonFungibleToken.CollectionPublic, CryptoPoops.CustomCollectionPublic}>()
                  ?? panic("This account does not have a Collection")

    return pubRef.borrowAuthNFT(id: id).name

}
```
