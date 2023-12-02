---
Name: Shreyas Pani
Topic: Floating point number systems
Title: Types of Floating Point Number Systems
---

# Types of Floating Point Number Systems

## Table of Contents
- [Introduction](#Introduction)
- [IEEE 754 System](#IEEE-754-System)
- [Non-IEEE Floating Point Number Systems](#Non-IEEE-Floating-Point-Number-Systems)
- [Applications for Different Types of Floating Point Number Systems](#Applications-for-Different-Types-of-Floating-Point-Number-Systems)
- [Implementation of Floating Point Number Systems](#Implementation-of-Floating-Point-Number-Systems)
- [Hardware Implications of Floating Point Number Systems](#Hardware-Implications-of-Floating-Point-Number-Systems)
- [Sources](#Sources)

## Introduction
The term "floating-point number" refers to any number which has a decimal point in some place. For example, 12.3, -43.2, and 50.0 are instances of floating point numbers, whereas numbers such as 1, 2, and -4 are not [1]. In the realm of computing, the term floating point number is used to describe any sort of number storage method that allows the number in question to have a decimal point. By far the most popular floating-point number system is the IEEE 754 method. However, some alternatives, such as posits, Bfloat16, minifloats, and fixed point arithmetic, exist with their own sets of pros and cons.

## IEEE 754 System

### Introduction to IEEE 754
By far the most popular method of storing floating-point numbers is through the IEEE 754 system. First invented in 1985, the IEEE 754 number system uses 3 main fields: a 1-bit sign field, an exponent field, and a mantissa field. The size of the exponent and mantissa field change based on the operating system. For example, in a 32-bit operating system (referred to as single-precision), the exponent is 8 bits and the mantissa is 23 bits, as shown below:

![](IEEE754Bits.jpg)[2]

However, in a 64-bit operating system (referred to as double-precision), the exponent is 11 bits and the mantissa is 52 bits. Note that the sign field will always only take up one bit, as one bit is sufficient to encode whether the number is positive or negative [3].

### Converting IEEE 754 to/from Decimal

Base-10 decimal numbers can be converted to IEEE 754 numbers and vice versa through the following formula:

$(-1)^{sign} * (1 + mantissa) * 2 ^ {exponent - bias}$, where $bias = 2 ^ {length \ of \ exponent} - 1$. Therefore, in a 32-bit system, bias = 127, and in a 64-bit system, bias = 1023. [4]

For example, if we had the 32-bit IEEE 754 floating point number 0xABC00000, we would have the following sign, exponent, and mantissa fields: \
sign = 1 \
exponent = 01010111 \
mantissa = 10000000000000000000000 \
bias = 127 \
Converting the exponent and mantissa to decimal yields: \
exponent = 87 \
mantissa = 0.5 (To convert the mantissa to decimal, multiply the leftmost number by $2^{-1}$, the next number by $2^{-2}$, and so on. Essentially, the mantissa can be treated as a number equal to 0.mantissa in base-2 form. In this case, that would be 0.10000000000000000000000 in base-2, which is equal to 0.5 in base-10)

This yields the expression $(-1)^{1} * (1 + 0.5) * 2 ^ {87 - 127}$ = $-1.5 * 2 ^ {-40}$ = $-1.36424 * 10 ^ {-12}$.

We can also do the opposite, converting from decimal to IEEE 754. To convert the number 3000 from decimal to 32-bit IEEE 754, first, an integer power of 2 has to be found such that $1 \geq \frac{3000}{2 ^ {exp}} < 2$. We can see that $2^{11} * 1.46484375 = 3000$. Therefore, we know our sign bit is 0 because 3000 is positive and our mantissa is 0.46484375. Our exponent will be equal to 11 + bias = 138. We are now left with these binary equivalents: \
sign = 0 \
exponent = 10001010 \
mantissa = 01110111000000000000 \
putting these numbers together yields an IEEE 754 representation of 01000101001110111000000000000, which is 45B80000 in hexadecimal notation.

### Special Number Representations

In addition to being able to model a very wide range of numbers, IEEE 754 also has a few special number representations for 0 and infinity. IEEE 754 has two forms of zero: positive and negative 0. This is because 0 is stored when the exponent and mantissa are 0. Therefore, a "positive" 0 is stored when the sign bit is 0, and a "negative" 0 is stored when the sign bit is 1. Infinity occurs when the exponent field is all ones (the largest possible exponent value) and the mantissa is 0. Similar to how zeroes are stored, a positive infinity is stored when the sign bit is 0, and a negative infinity is stored when the sign bit is 1. NaN (not a number) is stored when the exponent field is all ones and the mantissa is some nonzero value; the sign bit can be either 0 or 1 for this case [3]. 

### IEEE 754 Pros and Cons

The most obvious upside of the IEEE 754 number system is the fact that it is so widely adopted. As a result, when creating low-level programs that need to be used on various types of computers, using the IEEE 754 system would be a very safe bet when compared to the other number systems discussed below. Another advantage of IEEE 754 is that it can display a very wide range of numbers. For example, not counting positive or negative infinity or zero, a 32-bit IEEE system can store numbers from 1.1754E-38 to 3.4028E+38, and the same range of negative numbers.

On the other hand, IEEE 754 also comes with its own set of downsides. The most prevalent downside of IEEE 754 is floating point error. When doing arithmetic operations on numbers that are several magnitudes away from each other (example: $10^{40} + 10^{-30}$), because of the way numbers are stored in IEEE 754, some digits will be cut off. This will lead to a loss of accuracy, especially if these errors are allowed to propagate through a certain problem and become larger [5]. Another downside of IEEE 754 is that it has a fixed number of bits for the mantissa and exponent. This could lead to unnecessary space being allocated to either field, which may potentially result in a loss of information because there are not enough bits to store the information [6].

## Non-IEEE Floating Point Number Systems

### Posits

Posits are arguably the most frequently cited alternative to IEEE 754 numbers. While the IEEE 754 system consists of 3 main parts: a sign bit, exponent, and mantissa, the posit instead has 4 parts: the sign bit, regime, exponent, and mantissa, as shown in the figure below: 

![](PositFormat.png)[7] \
\* Note that this figure calls the mantissa portion of the number a fraction - both terms are interchangeable in this context. Also, as will be discussed shortly, the size of the exponent field is variable and does not necessarily have to be 2. 

The regime bits can vary in length based on the size of the number. The length of the regime is determined by the length of consecutive zeros or 1s. For example, if directly after the sign bit we had 0001, the regime is considered to be 000. Conversely, if we had 1111110 in the regime space, the regime is considered to be 111111. In the example above, there are two exponent bits. However, this can change based on the implementation used. The number $es$ represents the maximum number of exponent bits possible (in the example above, $es = 2$). If there are at least $es$ bits left after the regime, then $es$ bits will be saved for the regime, and any leftover bits will be the mantissa portion of the number. If there are $es$ or less bits left, then all of the remaining bits will be dedicated to the exponent, and no bits will be allocated for the mantissa. 

To convert between posit numbers to base-10 (and vice versa), the following formula is used:

$$num = (-1)^b * (1 + mantissa) * 2^{e + k * 2^{es}}$$[8]

Where b is the sign bit, mantissa is the value inside the mantissa (calculated the same way as in IEEE 754 format), and e is the value in the exponent field. k is defined as the length of the regime space. If there are $l$ 1s in the regime space, k = $l - 1$. If there are instead $l$ 0s in the regime space, then k = $-l$. Lastly, es is defined as the maximum number of exponent bits possible. 

There are two special numbers in the posit format: 0 and +/- infinity. When the whole posit number is 0s, then zero is stored. When a 1 is followed by all 0s, then the number is stored as +/- infinity. There is no way to store a strictly positive or strictly negative infinity in the posit format. [9]

The main upside to posit numbers is that they are more accurate for numbers with exponents close to 0. This is because more bits can be dedicated to the fraction portion of the number since we will likely have a regime value close to 0. Posits also have a much greater range than a similar size IEEE number thanks to the inclusion of the regime field.

![](PositAccuracy.png)[7]

The main drawback to posit numbers is shown in the graph above; while Posits are slightly more accurate with numbers that have an exponent near 0, they are much more inaccurate for very large or very small numbers. This is because fewer bits are dedicated to the fraction in posit numbers when compared to IEEE numbers if several regime bits are needed, leading to a loss in information.

### Bfloat16

Bfloat16 is a number format designed specifically for machine learning applications. As the name suggests, it is a 16-bit number format that has 1 sign bit, 7 exponent bits, and 8 mantissa bits. 

![](bFloat16Format.png)[10]

Calculations to convert bfloat16 to decimal and vice versa are done in the exact same manner as IEEE 754 numbers: $(-1)^{sign} * (1 + mantissa) * 2 ^ {exponent - bias}$. However, in this case, because the size of the exponent field in bfloat16 numbers is fixed, the bias term will always be equal to 127. Therefore, we can simply the formula to $(-1)^{sign} * (1 + mantissa) * 2 ^ {exponent - 127}$. 

The main upside to bfloat16 is hardware efficiency. Because bfloat16 numbers only have 16 bits, hardware computations can be done much faster than with 32 or 64-bit numbers. One such application is the realm of machine learning, where massive amounts of computations have to be completed to output a result. 

The most prevalent downside of bfloat16 also comes as a result of its size. Because bfloat16 is only 16 bits, it cannot store nearly as much information as a 32-bit IEEE or posit number can. In other words, the tradeoff for faster computations with bfloat16 numbers is a lower accuracy.

### Minifloats

Minifloats are a more general type of bfloat16 numbers. The term minifloat refers to any floating-point number represented with very few bits. The format of these numbers is described using a list of 4 values: (S, E, M, B). S is the length of the sign field, which is almost always either 0 or 1. E is the length of the exponent field, M is the length of the mantissa, and B is the value of the bias term in converting the numbers from binary to decimal [11]. For example, if a minifloat had the format (1, 5, 6, 31), the binary number would be 1 + 5 + 6 = 12 bits long, and to convert these binary numbers to decimal and vice versa, the formula would be $(-1)^{sign} * (1 + mantissa) * 2 ^ {exponent - 31}$. 

Minifloats have very similar pros and cons to bfloat16 numbers; their smaller size allows for faster computations, but this smaller size comes at the cost of accuracy.

### Fixed-Point Numbers

Fixed-point numbers are a type of floating point number which has a fixed number of bits dedicated to the left of the decimal point, and a fixed number of bits dedicated to the right of the decimal point. The format of these numbers is denoted as U(x, y), where x is the number of bits to the left of the decimal place (the integer portion of the number), and y is the number of bits to the right of the decimal place (the fractional portion of the number). For example, a U(10, 6) number would look like:


## Applications for Different Types of Floating Point Number Systems

### IEEE 754

### Posits

### Bfloat16 and Minifloats
Machine Learning [10]
Computer Vision [12]

### Fixed-Point Numbers

## Implementation of Floating Point Number Systems

## Hardware Implications of Floating Point Number Systems

## Sources
1. https://www.freecodecamp.org/news/floating-point-definition/
2. https://www.geeksforgeeks.org/ieee-standard-754-floating-point-numbers/
3. https://docs.oracle.com/cd/E19957-01/806-3568/ncg_math.html#:~:text=IEEE%20754%20specifies%3A,and%20occupies%2064%20bits%20overall.
4. https://mathcenter.oxford.emory.edu/site/cs170/ieee754/
5. https://learn.microsoft.com/en-us/office/troubleshoot/excel/floating-point-arithmetic-inaccurate-result
6. https://babbage.cs.qc.cuny.edu/IEEE-754.old/References.xhtml
7. https://spectrum.ieee.org/floating-point-numbers-posits-processor
8. https://www.johndcook.com/blog/2018/04/11/anatomy-of-a-posit-number/#:~:text=A%20posit%20number%20type%20is,devoted%20to%20the%20exponent%2C%20es
9. https://www.sigarch.org/posit-a-potential-replacement-for-ieee-754/
10. https://cloud.google.com/tpu/docs/bfloat16
11. https://mrob.com/pub/math/floatformats.html
12. https://www.mdpi.com/2076-3417/11/23/11164
