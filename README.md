<h1 align="center">bloom.mq</h1>

> [!WARNING]
> This project is currently under active development. APIs and features may change without notice.

A Bloom filter for [mq](https://github.com/harehare/mq), as a module: a compact set that answers "definitely not in the set" or "probably in the set". Use it to skip work for items you have already seen, such as deduplicating a stream of lines or URLs.

## Installation

Copy `bloom.mq` to your mq module directory, or place it anywhere and reference it with `-L`.

```sh
cp bloom.mq ~/.local/mq/config/
```

HTTP imports are disabled by default; pass `--allow-http-import` to import directly from GitHub:

```sh
mq --allow-http-import -I null 'import "github.com/harehare/bloom.mq" | bloom::bits(bloom::for_capacity(1000, 0.01))'
```

## Usage

```sh
mq -L . -I null '
  import "bloom"
  | let bf = bloom::add_all(bloom::for_capacity(1000, 0.01), ["apple", "banana", "cherry"])
  | [bloom::contains(bf, "apple"), bloom::contains(bf, "durian"), bloom::count(bf)]
'
# [true, false, 3]
```

A `true` from `contains` may be wrong, at about the rate you asked for. A `false` never is.

| Function | Description |
| --- | --- |
| `for_capacity(capacity, error_rate)` | empty filter sized for `capacity` items at about `error_rate`, for example `1000, 0.01` |
| `new(bits, hashes)` | empty filter with explicit size |
| `add(bf, item)` / `add_all(bf, items)` | add one item or an array of items |
| `contains(bf, item)` | whether the item may have been added |
| `union(a, b)` | filter holding the items of both, which must have the same `bits` and `hashes` |
| `count(bf)` | number of adds, counting an item again each time it is added |
| `fill_ratio(bf)` | fraction of bits set |
| `estimated_error_rate(bf)` | false positive rate to expect from the bits set now |
| `bits(bf)` / `hashes(bf)` | the filter's size and bits set per item |

## Notes

- Items are compared by the string `to_string` gives for them, so `42` and `"42"` are the same item.
- A filter is a plain array and every operation returns the updated filter. Write `bf = bloom::add(bf, x)` and do not read the old `bf` afterwards, so the update happens in place.
- Items cannot be removed, and filters cannot be intersected.
- Each item sets `hashes` bits chosen from its SHA-256 digest by double hashing. Bits are packed 32 to a number.
- `add` and `contains` cost O(`hashes`). `fill_ratio` and `union` cost O(`bits`).

## Development

```sh
mq-test bloom_tests.mq
mq-bench -L . bloom_bench.mq
```
