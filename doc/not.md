% What #[derive(Not)] generates

# Tuple structs

When deriving for a tuple struct with two fields like this:

```
#[derive(Not)]
struct MyInt(i32, i32)
```

Code like this will be generated:

```
impl ::std::ops::Not for MyInts {
    type Output = MyInts;
    fn not(self) -> MyInts {
        MyInts(self.0.not(), self.1.not())
    }
}
```

The behaviour is similar with more or less fields.


# Regular structs

When deriving for a regular struct with two fields like this:

```
#[derive(Not)]
struct Point2D {
    x: i32,
    y: i32,
}
```

Code like this will be generated:

```
impl ::std::ops::Not for Point2D {
    type Output = Point2D;
    fn not(self) -> Point2D {
        Point2D {
            x: self.x.not(),
            y: self.y.not(),
        }
    }
}
```

The behaviour is similar with more or less fields.


# Enums

For each enum variant `Not` is derived in a similar way as it would be derived
if it would be its own type.
For instance when deriving `Not` for an enum like this:

```
#[derive(Not)]
enum MixedInts {
    SmallInt(i32),
    BigInt(i64),
    TwoSmallInts(i32, i32),
    NamedSmallInts { x: i32, y: i32 },
    UnsignedOne(u32),
    UnsignedTwo(u32),
}
```

Code like this will be generated:

```
impl ::std::ops::Not for MixedInts {
    type Output = MixedInts;
    fn not(self) -> MixedInts {
        match self {
            MixedInts::SmallInt(__0) => MixedInts::SmallInt(__0.not()),
            MixedInts::BigInt(__0) => MixedInts::BigInt(__0.not()),
            MixedInts::TwoSmallInts(__0, __1) => MixedInts::TwoSmallInts(__0.not(), __1.not()),
            MixedInts::NamedSmallInts { x: __0, y: __1 } => {
                MixedInts::NamedSmallInts {
                    x: __0.not(),
                    y: __1.not(),
                }
            }
            MixedInts::UnsignedOne(__0) => MixedInts::UnsignedOne(__0.not()),
            MixedInts::UnsignedTwo(__0) => MixedInts::UnsignedTwo(__0.not()),
        }
    }
}
```

There is one important thing to remember though.
If you add a unit variant to the enum its return type will change from
`EnumType` to `Result<EnumType>`.
This is because Unit cannot have `Not` implemented.
So, when deriving `Not` for an enum like this:

```
#[derive(Not)]
enum EnumWithUnit {
    SmallInt(i32),
    Unit,
}
```

Code like this will be generated:

```
impl ::std::ops::Not for EnumWithUnit {
    type Output = Result<EnumWithUnit, &'static str>;
    fn not(self) -> Result<EnumWithUnit, &'static str> {
        match self {
            EnumWithUnit::SmallInt(__0) => Ok(EnumWithUnit::SmallInt(__0.not())),
            EnumWithUnit::Unit => Err("Cannot not() unit variants"),
        }
    }
}
```