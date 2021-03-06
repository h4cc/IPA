# IPA

A pale, hoppy library for working with Internet Protocol Addresses.

> Currently only supports individual addresses and not networks, and only
> classic IPv4, not modern IPv6, sorry.

## Installation

If [available in Hex](https://hex.pm/docs/publish), the package can be installed as:

  1. Add ipa to your list of dependencies in `mix.exs`:

        def deps do
          [{:ipa, "~> 0.0.2"}]
        end

  2. Ensure ipa is started before your application:

        def application do
          [applications: [:ipa]]
        end

## Usage

The following functions are available in the `IPA` module:

Check if a dotted decimal, dotted binary, hexadecimal, binary
or 4-element tuple IP address is valid:

```elixir
iex> IPA.valid_address?("192.168.0.1")
true
iex> IPA.valid_address?("192.168.0.256")
false
iex> IPA.valid_address?("192.168.0")
false
iex> IPA.valid_address?("192.168.0.1.1")
false
iex> IPA.valid_address?("11000000.10101000.00000000.00000001")
true
iex> IPA.valid_address?("0xC0A80001")
true
iex> IPA.valid_address?("0b11000000101010000000000000000001")
true
iex> IPA.valid_address?({192, 168, 0, 1})
true
```

> This validity is slightly imperfect as it does not recognise
> the fact that 127.1 can be considered a valid IP address
> that translates to 127.0.0.1.

Check if a dotted decimal, dotted binary or CIDR notation subnet mask is valid:

```elixir
iex> IPA.valid_mask?("255.255.255.0")
true
iex> IPA.valid_mask?("192.168.0.1")  
false
iex> IPA.valid_mask?(24)             
true
iex> IPA.valid_mask?(33)             
false
iex> IPA.valid_mask?("11111111.11111111.11111111.00000000")
true
iex> IPA.valid_mask?("10101000.10101000.00000000.00000000")
false
iex> IPA.valid_mask?("0xFFFFFF00")
true
iex> IPA.valid_mask?("0b11111111111111111111111100000000")
true
iex> IPA.valid_mask?({255, 255, 255, 0})
true
```

Transform an address or mask between dotted decimal, dotted binary, hexadecimal, binary, 4-element tuple and CIDR representation:

```elixir
# IPA.to_bits
# dotted decimal -> dotted binary
iex> IPA.to_bits("192.168.0.1")
"11000000.10101000.00000000.00000001"
# hexadecimal -> dotted binary
iex> IPA.to_bits("0xC0A80001")
"11000000.10101000.00000000.00000001"
# tuple -> dotted binary
iex> IPA.to_bits({192, 168, 0, 1})
"11000000.10101000.00000000.00000001"
# binary -> dotted binary
iex> IPA.to_bits("0b11000000101010000000000000000001")
"11000000.10101000.00000000.00000001"
# cidr -> dotted binary
iex> IPA.to_bits(24)
"11111111.11111111.11111111.00000000"
# invalid address
iex> IPA.to_bits("192.168.0.256")
** (IPError) Invalid IP Address

# IPA.to_hex
# dotted decimal -> hexadecimal
iex> IPA.to_hex("192.168.0.1")
"0xC0A80001"
# binary -> hexadecimal
iex> IPA.to_hex("0b11000000101010000000000000000001")
"0xC0A80001"
# dotted binary -> hexadecimal
iex> IPA.to_hex("11000000.10101000.00000000.00000001")
"0xC0A80001"
# tuple -> hexadecimal
iex> IPA.to_hex({192, 168, 0, 1})
"0xC0A80001"
# cidr -> hexadecimal
iex> IPA.to_hex(24)
"0xFFFFFF00"
# invalid addresses
iex> IPA.to_hex("192.168.0.256")
** (IPError) Invalid IP Address
iex> IPA.to_hex("0b11000000101010000000000000000002")
** (IPError) Invalid IP Address

# IPA.to_binary
# dotted decimal -> binary
iex> IPA.to_binary("192.168.0.1")
"0b11000000101010000000000000000001"
# hexadecimal -> binary
iex> IPA.to_binary("0xC0A80001")
"0b11000000101010000000000000000001"
# dotted binary -> binary
iex> IPA.to_binary("11000000.10101000.00000000.00000001")
"0b11000000101010000000000000000001"
# tuple -> binary
iex> IPA.to_binary({192, 168, 0, 1})
"0b11000000101010000000000000000001"
# cidr -> binary
iex> IPA.to_binary(24)
"0b11111111111111111111111100000000"
# invalid address
iex> IPA.to_binary({192, 168, 0, 256})
** (IPError) Invalid IP Address

# IPA.to_octets
# dotted decimal address -> tuple
iex> IPA.to_octets("192.168.0.1")
{192, 168, 0, 1}
# dotted decimal mask -> tuple
iex> IPA.to_octets("255.255.255.0")
{255, 255, 255, 0}
# binary -> tuple
iex> IPA.to_octets("0b11000000101010000000000000000001")
{192, 168, 0, 1}
# hexadecimal -> tuple
iex> IPA.to_octets("0xC0A80001")
{192, 168, 0, 1}
# dotted binary -> tuple
iex> IPA.to_octets("11000000.10101000.00000000.00000001")
{192, 168, 0, 1}
# invalid address
iex> IPA.to_octets("192.168.0.256")
** (IPError) Invalid IP Address

# IPA.to_dotted_dec
# cidr -> dotted decimal
iex> IPA.to_dotted_dec(24)
"255.255.255.0"
# tuple -> dotted decimal
iex> IPA.to_dotted_dec({192, 168, 0, 1})
"192.168.0.1"
# binary -> dotted decimal
iex> IPA.to_dotted_dec("0b11000000101010000000000000000001")
"192.168.0.1"
# hexadecimal -> dotted decimal
iex> IPA.to_dotted_dec("0xC0A80001")
"192.168.0.1"
# dotted binary -> dotted decimal
iex> IPA.to_dotted_dec("11000000.10101000.00000000.00000001")
"192.168.0.1"
# invalid mask/address
iex> IPA.to_dotted_dec(33)
** (SubnetError) Invalid Subnet Mask
iex> IPA.to_dotted_dec({192, 168, 0, 256})
** (IPError) Invalid IP Address

# IPA.to_cidr
# dotted decimal to cidr
iex> IPA.to_cidr("255.255.255.0")
24
# hexadecimal -> cidr
iex> IPA.to_cidr("0xFFFFFF00")
24
# binary -> cidr
iex> IPA.to_cidr("0b11111111111111111111111100000000")
24
# tuple -> cidr
iex> IPA.to_cidr({255, 255, 255, 0})
24
# invalid mask
iex> IPA.to_cidr("192.168.0.1")
** (SubnetError) Invalid Subnet Mask
```

Find out if the address is part of a reserved private address block
(ie. NOT a public address):

```elixir
iex> IPA.reserved?("192.168.0.1")
true
iex> IPA.reserved?("8.8.8.8")    
false
```

Find out which reserved block of addresses an address is a part of.
If not reserved it will be public, obvs.  A full list of reserved
blocks can be found in the docs:

```elixir
iex> IPA.block("192.168.0.1")
:rfc1918
iex> IPA.block("10.0.1.0")   
:rfc1918
iex> IPA.block("127.0.0.1")  
:loopback
iex> IPA.block("8.8.8.8")    
:public
```

## Tests

Run `mix test`. DocTests included.

## Docs are available on [hexdocs](https://hexdocs.pm/ipa/api-reference.html) or by running `mix docs`.

## Caveat

Honestly, I don't know how much time I'm actually going to be able to dedicate
to this project, so maybe think carefully before using it in production code
(ha, high hopes!), but perhaps it can at least provide a starting point for
something better?
