1. `.link()` simply links data from `/storage/` to either `/public/` or `/private/` path in the account storage.
2. The resource interface is like a blueprint for the resource. So if you wanted everyone to only be able to see certain aspects of a resource then this is where you could link the resource stored in `/storage/` with the resource that implement the resource interface to `/public/` to only show those attributes listed in the resource interface.
3.

```
pub contract Properties {

    pub var properties: @{UInt64: Property}

    pub resource interface IProperty {
        pub let acres: UFix64
        pub let priceInUSD: UFix64
        pub let state: String
        pub let city: String
    }

    pub resource Property: IProperty {
        pub let acres: UFix64
        pub let priceInUSD: UFix64
        pub let state: String
        pub let city: String
        pub let seller: String

        init(_acres: UFix64, _priceInUSD: UFix64, _state: String, _city: String, _seller: String) {
            self.acres = _acres
            self.priceInUSD = _priceInUSD
            self.state = _state
            self.city = _city
            self.seller = _seller
        }
    }

    pub fun createProperty(acres: UFix64, priceInUSD: UFix64, state: String, city: String, seller: String): @Property {
        let property: @Property <- create Property(_acres: acres, _priceInUSD: priceInUSD, _state: state, _city: city, _seller: seller)
        return <- property
    }

    pub fun getReference(key: UInt64): &Property {
        return &self.properties[key] as &Property
    }

    init() {
        self.properties <- {
            1: <- create Property(_acres: 0.84, _priceInUSD: 3500000.00, _state: "Georgia", _city: "Atlanta", _seller: "Shain")
        }
    }
}
```

i.

```
import Properties from 0x01

transaction(acres: UFix64, priceInUSD: UFix64, state: String, city: String, seller: String) {

  prepare(signer: AuthAccount) {
    let niceProperty <- Properties.createProperty(acres: acres, priceInUSD: priceInUSD, state: state, city: city, seller: seller)
    signer.save(<- niceProperty, to: /storage/MyNiceProperty)
    signer.link<&Properties.Property{Properties.IProperty}>(/public/MyNiceProperty, target: /storage/MyNiceProperty)

  }

  execute {

  }
}

```

ii.

```
import Properties from 0x01

pub fun main(address: Address): String {
  let publicCapability: Capability<&Properties.Property{Properties.IProperty}> =
    getAccount(address).getCapability<&Properties.Property{Properties.IProperty}>(/public/MyNiceProperty)

  let badProperty: &Properties.Property{Properties.IProperty} = publicCapability.borrow() ?? panic("The capability doesn't exist or you did not specify the right type when you got the capability.")

  return badProperty.seller
}

```

iii.

```
import Properties from 0x01

pub fun main(address: Address): UFix64 {
  let publicCapability: Capability<&Properties.Property{Properties.IProperty}> =
    getAccount(address).getCapability<&Properties.Property{Properties.IProperty}>(/public/MyNiceProperty)

  let badProperty: &Properties.Property{Properties.IProperty} = publicCapability.borrow() ?? panic("The capability doesn't exist or you did not specify the right type when you got the capability.")

  return badProperty.acres
}

```
