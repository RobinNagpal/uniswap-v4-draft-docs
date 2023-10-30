We have a pool of USDC and ETH

Assume that the pool as 0.3% fee
tickSpacing = 60

Some basic maths programs

```python
import math

def calculate_tick(price):
    tick = int(math.log(price / math.sqrt(1.0001)) / math.log(1.0001))
    return tick

# Example usage
price = 900.0  # Replace with your price
tick = calculate_tick(price)
print(f"The tick position for a price of {price} is {tick}")

reverse_price = 1.0001 ** tick

print(f"reverse price {reverse_price}")

def calculate_sqrt_price_x96(price):
    sqrt_price_x96 = int(math.sqrt(price) * 2**96)
    return sqrt_price_x96

def calculate_price_from_sqrt_price_x96(sqrt_price_x96):
    price = (sqrt_price_x96 / (2**96)) ** 2
    return price
    
# Example usage
price = 1000.0  # Replace with your price
sqrt_price_x96 = calculate_sqrt_price_x96(price)
print(f"The sqrt(price) * 2^96 for a price of {price} is {sqrt_price_x96}")

original_price = calculate_price_from_sqrt_price_x96(sqrt_price_x96)
print(f"original price {original_price}")
```


1) We deposit 2000 USDC and 1 ETH in the range of 1800 to 2200

using the below code lets calculate tickLower for 2000

The tick position for a price of 1800.0 is 74958
The tick position for a price of 2000.0 is 76012
The tick position for a price of 2200.0 is 76965

The sqrt(price) * 2^96 for a price of 1800.0 is 3361366258487168519347365740544
The sqrt(price) * 2^96 for a price of 2000.0 is 3543191142285914378072636784640
The sqrt(price) * 2^96 for a price of 2200.0 is 3716130220787573499654546915328

