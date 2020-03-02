# Yul Plus

[![npm version](https://badge.fury.io/js/yulp.svg)](https://badge.fury.io/js/yulp)

A low-level, highly efficient extension to Yul, an intermediate smart-contract language for the Ethereum Virtual Machine.

## Features
- All existing Yul features
- Memory structures (`mstruct`)
- Enums (`enum`)
- Constants (`const`)
- Ethereum Standard ABI Signature Generation (`sig"function ..."`)
- Memory slice (`mslice`)
- Booleans (`true`, `false`)

## Coming Soon
- Safe Math
- Types
- Import/Export

## Install

```sh
npm install --save yulp
```

## Usage

```js
const yulp = require('../index');
const source = yulp.compile(`
object "SimpleStore" {
  code {
    datacopy(0, dataoffset("Runtime"), datasize("Runtime"))
    return(0, datasize("Runtime"))
  }
  object "Runtime" {
    code {
      calldatacopy(0, 0, 36)
      switch mslice(0, 4) // new memory slice method

      case sig"function store(uint256 val)" { // new signature method
        sstore(0, mload(4))
      }

      case sig"function get() returns (uint256)" {
        mstore(100, sload(0))
        return (100, 32)
      }
    }
  }
}
`);

console.log(yulp.print(source.results));
```

## Enums

Here we have a fully featured scoped `enum` identifier which acts as a constant.

```
object "contract" {
  code {
    enum Colors (
      Red,
      Blue,
      Green
    )

    log1(0, 0, Colors.Green) // logs 2 as a topic
  }
}
```

## Constants

`const` will define a `let` variable value that cannot be changed.

```
object "contract" {
  code {
    const firstVar = 0xaa
    const someOther = 1
  }
}
```

## Memory Slice

`mslice(position, length)` will return a 1-32 byte value from memory.

```
object "contract" {
  code {
    mstore(30, 0xaaaa)

    log1(0, 0, mslice(30, 2)) // will log 0xaaaa
  }
}
```

## Booleans

`true` and `false` are added and equate to values `0x01` and `0x00`.

```
object "contract" {
  code {
    mstore(30, true)

    log1(0, 0, mload(30)) // will log 0x01
  }
}
```

## Ethereum Standard ABI Signature and Topic Generation

`sig" [ method abi ] "` will equate to a 4 byte method signature hex value

`topic" [ event abi definition ] "` will equate to the 32 byte topic hash

```
object "contract" {
  code {
    const storeSig = sig"function store(uint256 val)"
    const eventSig = topic"event Store (uint256 indexed val)"

    log1(0, 0, storeSig) // will log 0x6057361d

    log1(0, 0, eventSig) // will log 0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef
  }
}
```

## Memory Structures

These memory descriptive structures allow virtual formation of in-memory structure like-data.

Below is an example of how many methods come packed in with memory structures.

They allow for data chunk sizes up to 32 bytes.

`mstruct` Identifier ( [ property, ... ] )

Property is defined by an identifier and length specifier (i.e. `blockProducer:32`) where the identifier is `blockProducer` and the length is `32` bytes.

Virtual memory array structure as defined as `[` item length `]` and specifier before hand a total length array `.length` ':' length size specifier.

```
object "contract" {
  code {
    // example structure in memory
    mstore(400, 0xaa) // block producer
    mstore(432, 0xbb) // previous block hash
    mstore(464, 0xcc) // block height
    mstore(496, 0x03) // length of anotherArray (i.e. 3 array items)
    mstore(554, 0xaaaabbbbcccc) // array with 3 elements, 0xaaaa, 0xbbbb, 0xcccc
    mstore(560, 0xdd) // Ethereum block number
    mstore(592, 0x01) // transaction roots length
    mstore(652, 0xffffffff) // transaction roots, one 4 byte item 0xffffffff

    mstruct BlockHeader (
      blockProducer: 32,
      previousBlockHash: 32,
      blockHeight: 32,
      anotherArray.length: 32,
      anotherArray: [2],
      ethereumBlockNumber: 32,
      transactionRoots.length: 32,
      transactionRoots: [4]
    )

    BlockHeader.blockProducer(400) // will return 0xaa
    BlockHeader.blockProducer.size() // will return 32

    BlockHeader.blockHeight(400) // will return block height 0xcc
    BlockHeader.blockHeight.offset(400) // will return pos + length
    BlockHeader.blockHeight.position(400) // will return pos
    BlockHeader.blockHeight.size() // 32
    BlockHeader.blockHeight.index() // 2

    BlockHeader.transactionRoots(400, 2) // will return array item 0xcccc

    BlockHeader.size(400) // will return the size of the entire struct
    BlockHeader.offset(400) // will return entire offset position of the struct pos + length
  }
}
```

## Help out

There is always a lot of work to do, and will have many rules to maintain. So please help out in any way that you can:

- Create, enhance, and debug fuel-core rules (see our guide to ["Working on rules"](./.github/CONTRIBUTING.md)).
- Improve documentation.
- Chime in on any open issue or pull request.
- Open new issues about your ideas for making `fuel-core` better, and pull requests to show us how your idea works.
- Add new tests to *absolutely anything*.
- Create or contribute to ecosystem tools.
- Spread the word!

We communicate via [issues](https://github.com/fuellabs/fuel-core/issues) and [pull requests](https://github.com/fuellabs/fuel-core/pulls).

## Important documents

- [Changelog](CHANGE_LOG.md)
- [License](https://raw.githubusercontent.com/fuellabs/fuel-core/master/LICENSE)

## Licence

This project is licensed under the Apache-2.0 license, Copyright (c) 2020 Fuel labs. For more information see LICENSE

```
Copyright 2020 Fuel Labs

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
