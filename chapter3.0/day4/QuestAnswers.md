1. The resource interface is essentially a blueprint for the resource. It specifies what is required and the access of these requirements for the resource.
2.

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

    pub fun getReference(key: UInt64): &Property {
        return &self.properties[key] as &Property
    }

    pub fun getPropertyPricePerSquareFoot(key: UInt64): UFix64 {
        let property = &self.properties[key] as &Property
        let totalSquareFeet = property.acres * 43560.0
        let pricePerSquareFoott = property.priceInUSD / totalSquareFeet
        return pricePerSquareFoott
    }

    pub fun getPropertySeller(key: UInt64): String {
        let property = &self.properties[key] as &Property{IProperty}
        return property.seller // ERROR: `member of restricted type is not accessible: seller`
    }

    init() {
        self.properties <- {
            1: <- create Property(_acres: 0.84, _priceInUSD: 3500000.00, _state: "Georgia", _city: "Atlanta", _seller: "Shain")
        }
    }
}
```

3.

```
pub contract Stuff {

    pub struct interface ITest {
      pub var greeting: String
      pub var favouriteFruit: String
      pub fun changeGreeting(newGreeting: String): String
    }

    pub struct Test: ITest {
      pub var greeting: String
      pub var favouriteFruit: String

      pub fun changeGreeting(newGreeting: String): String {
        self.greeting = newGreeting
        return self.greeting // returns the new greeting
      }

      init() {
        self.greeting = "Hello!"
        self.favouriteFruit = "Strawberry!"
      }
    }

    pub fun fixThis() {
      let test: Test{ITest} = Test()
      let newGreeting = test.changeGreeting(newGreeting: "Bonjour!")
      log(newGreeting)
    }
}
```
