1. 
```
pub contract Properties {
    pub var properties: @[Property]

    pub resource PropertyBucket {
        pub var propertyBucket: @{UInt64: Property}

        pub fun addProperty(property: @Property) {
            self.propertyBucket[property.uuid] <-! property
        }

        pub fun removePropertya(uuid: UInt64): @Property {
            let property <- self.propertyBucket.remove(key: uuid) ?? panic("Could not find the greeting!")
            return <- property
        }

        init() {
            self.propertyBucket <- {}
        }

        destroy() {
            destroy self.propertyBucket
        }
    }

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

    pub fun createPropertyBucket(): @PropertyBucket {
        let propertyBucket: @PropertyBucket <- create PropertyBucket()
        return <- propertyBucket
    }

    pub fun addProperty(property: @Property) {
        self.properties.append(<- property)
    }

    pub fun removeProperty(index: Int): @Property {
        return <- self.properties.remove(at: index)
    }

    init() {
        self.properties <- []
    }
}
```