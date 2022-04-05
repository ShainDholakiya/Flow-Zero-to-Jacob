1. All the smart contracts a user has deployed and all their account storage lives in their account on Flow.
2.

- `/storage/` - this is where all data is stored and can only be accessed by the account owner
- `/public/` - data available to anyone
- `/private/` - data only available to account owner and people that are given access to by the owner

3.

- `.save()` - adds data to whatever account storage path you give it
- `.load()` - take out the data from whatever account storage path you give it
- `.borrow()` - simply read the data from whatever account storage path you give it

4. Bcause we do not have access to `AuthAccount` which is specific to the signer so we would not know who to save the data to and saving data would need a transition cost associated with it.
5. The account owner would have to sign the transaction for the data to save in their account so I would have to sign it myself for it to work.
6.

```
pub contract Properties {

    pub var properties: @{UInt64: Property}

    pub resource Property {
        pub let acres: UFix64
        pub let priceInUSD: UInt64
        pub let state: String
        pub let city: String

        init(_acres: UFix64, _priceInUSD: UInt64, _state: String, _city: String) {
            self.acres = _acres
            self.priceInUSD = _priceInUSD
            self.state = _state
            self.city = _city
        }
    }

    pub fun createProperty(acres: UFix64, priceInUSD: UInt64, state: String, city: String): @Property {
        let property: @Property <- create Property(_acres: acres, _priceInUSD: priceInUSD, _state: state, _city: city)
        return <- property
    }

    pub fun getReference(key: UInt64): &Property {
        return &self.properties[key] as &Property
    }

    init() {
        self.properties <- {
            1: <- create Property(_acres: 0.84, _priceInUSD: 3500000, _state: "Georgia", _city: "Atlanta")
        }
    }
}
```

i.

```
import Properties from 0x05

transaction(acres: UFix64, priceInUSD: UInt64, state: String, city: String) {

  prepare(signer: AuthAccount) {
    let niceProperty <- Properties.createProperty(acres: acres, priceInUSD: priceInUSD, state: state, city: city)
    signer.save(<- niceProperty, to: /storage/MyNiceProperty)
    let badProperty <- signer.load<@Properties.Property>(from: /storage/MyNiceProperty)
                          ?? panic("A `@Properties.Property` resource does not live here.")
    log(badProperty.acres)

    destroy badProperty
  }

  execute {

  }
}
```

ii.

```
import Properties from 0x05

transaction(acres: UFix64, priceInUSD: UInt64, state: String, city: String) {

  prepare(signer: AuthAccount) {
    let niceProperty <- Properties.createProperty(acres: acres, priceInUSD: priceInUSD, state: state, city: city)
    signer.save(<- niceProperty, to: /storage/MyNiceProperty)
    let goodProperty = signer.borrow<&Properties.Property>(from: /storage/MyNiceProperty)
                          ?? panic("A `@Properties.Property` resource does not live here.")
    log(goodProperty.acres)

  }

  execute {

  }
}
```
