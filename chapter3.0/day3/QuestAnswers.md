1. 
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
2. 
```
import Properties from 0x05

pub fun main(): UFix64 {
    let ref = Properties.getReference(key: 1)
    return ref.acres
}
```
3. References can save a lot of developer time in both coding and understanding the code. This is because if the program is moving around a resource so much, it might get confusing to read. References allow us to read or update resources without actually having to move them around. We can just pass in a reference of that piece of data instead of the actual data itself.