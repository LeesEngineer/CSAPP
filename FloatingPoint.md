</br>

# Representable Numbers

</br>

<p>There are two limitations to using binary to reprensent floating point numbers:</p>

<p><b>Limitation #1</b> : Can only exactly represent numbers of the form x / 2^k，<b>Other rational numbers have repeating bit representations</b>.For example:</p>

```
Value      Representation
1 / 3      0.010101010101[01]...
1 / 5      0.001100110011[0011]...
1 / 10     0.0001100110011[0011]...
```

<p><b>Limitation #2</b> : Just one setting of binary point within the w bits. Limited range of numbers</p>

<p>There's only so many bits to the left and the right of the binary point. So if we move the binary point to the left. Then we can't represent as many large numbers. We can only represent small numbers. But we have more precision to the right of the binary point. So we can represent more fractional values. Just the range of those values will be mush smaller</p>

<p>Similiar, if we move the binary to the right. We'll have a larger range of values.</p>

<p>Therefore, floating point is chosen as a compromise representation method</p>

<p>So, floating point is a representation to try that enables us to move that binary point to represent sort of as wide a range as possible, with as much precision given the number of bits</p>

<p>So the floating point is this sort of shifting binary point</p>

</br>

# Floating Point Representation

</br>

<p><b>Numerical Form</b>: `(-1)^s * M * 2^E`</p>

- Sign bit s determines whether number is negative or positive

- Significand M (mantissa) normally a fractional value in range [1.0, 2.0)

- Exponent E weights value by power of two

<p>It represents numbers in a way like a scientific notation. All of the numbers that we can represent in floating point have to be represented in this form</p> 

<p><b>Encoding</b>:(in either 32-bit or 64-bit)</p>

- MSB S is sign bit s

- exp field encodes E (but is not equal to E)

- frac field encodes M (but is not equal to E)

![14ad95ea3ef21c47c6f6b8849fa91a66](https://github.com/user-attachments/assets/d2584ae2-986b-41ad-a89b-c6d2df706dd0)

<p><b>Precision options</b></p>

```
Single precision: 32 bits
s     -  1-bits
exp   -  8-bits
frac  -  32-bits
Single precision: 64 bits
s     -  1-bits
exp   -  11-bits
frac  -  52-bits
Extended precision: 80 bits (Intel only)
s     -  1-bits
exp   -  15-bits
frac  -  63 or 64-bits
```

</br>

## Normalized Values

</br>

<p>When the exponent of a floating-point number is neither all 0s nor all 1s, the number is a normalized number.</p>

- exp != 000...0 and exp != 111...1.

- Exponent coded as a biased value E = exp - bias

  1. exp: unsigned value of exp field
 
  2. bias = 2^(k-1) - 1, where k is number of exponent bits. Single precision : (exp: 1...254, E: -126...127, bias: 127). Double precision : (exp: 1...2046, E: -1022...1023, bias: 1023).

<p>We've already learned about two's complement. That's a perfectly fine way to represent positive and negative numbers. We have exponents that are negative and positive. So why not just use a two's complement in the exp field to represent those positive and negative exponents.</p>

<p>Think about this and we'll come back to it. If we encode the exponent E using this bias representation. The smallest negative exponent is representated by all zeros, And the largest exponent is representated by 01...111</p>

<p>By using this biased representation, we can just compare two floating-point numbers just as unsigned. We can treat the whole floating-point number as an unsigned integer and compare two number. And get a true comparison</p> 

- Significand coded with implied leading 1: M = 1.xxx...x_2

  1. xxx...x : bits of frac field
 
  2. Minimum when frac = 000...0 (M = 1.0)
 
  3. Maximum when frac = 111...1 (M = 2.0 - epsilon)
 
  4. Get extra leading bit '1' for free

<p>We are always going to normalize M as 1.xxx...x</p>

```
float F = 15213.0;

15213_10 = 11101101101101_2
         = 1.1101101101101_2 * 2^13

significand
M = 1.1101101101101_2
frac = 11011011011010000000000_2
// We pat it out with zeros to get the 23 bits that we need for single precision

Exponent
E = 13
Bias = 127
Exp = 140 = 10001100_2

result:
0 10001100 11011011011010000000000
s exp      frac
```

</br>

## Normalized Values

</br>

<p>Now these normalized values always have this implied one. When we want to represent numbers closer to zero. That limits us. So there is another type of floating-point number called the denormalized value which is characterized by an exp field of all 0. And in a denormalized number there's no implied one. Denorms allow us to represent values that are very close to zero.</p>

<p>Why it is designed like this. Because we want floating point numbers to be "continuously close to 0". If all numbers are required to have an implied 1, then the smallest normalized number is: 1.0^{-127}. Any number smaller than this will become 0 directly, resulting in a "precision gap"</p>

```
... → 1.000000000 × 2^-126 → 0
```

<p>When exponent = 000…0, leading 1s are no longer assumed, but leading 0s are.</p>

- Condition: exp = 000...0

- Exponent value: E = 1 - Bias (Instead of E = 0 - bias). Because this can maintain the continuity of the value and prevent the normalization number from being discontinuous.

- Significand coded with implied leading 0:M = 0.xxx...x_2

- Cases

  1. exp = 000...0, frac = 000...0 : Represents zero value. Note distinct values: +0 and -0 (sign bit)
 
  2. exp = 000...0 ,frac != 000...0 : Numbers closest to 0.0. Equispaced

<p>This design ensures that floating-point numbers maintain a smooth distribution around 0, rather than abruptly jumping from the minimum normalized value to 0.</p>

</br>

## Special Values

</br>

- Condition: exp = 111...1

- Case: exp = 111...1, frac = 000...0

  1. represents value infinity
 
  2. Operation that overflows
 
  3. Both positive and negative
 
  4. 1.0 / 0.0 = -1.0 / -0.0 = positive infinity; 1.0 / -0.0 = negative infinity
 
<p>In floating-point we just overflow to the sticky value called infinity, and then everything we do on that remains infinity.</p>
 
- Case: exp = 111...1, frac != 000...0

  1. Not a number (NaN)
 
  2. Represents case when no numeric value can be determined
 
  3. sqrt(-1), infinity - infinity, infinity * 0
 
<p></p>




























































































































































































































































































































































































































































































































































