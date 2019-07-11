---
layout: post
title: Using Quaternions for rotations in $\mathbb{R}^3$
snippet: Quaternions exist in a four-dimensional complex hyperspace, and can be used to perform rotations in $\mathbb{R}^3$
tags: [computer programming, math, julia]
author: Arthur York
---

Quaternions are four-dimensional complex numbers of the form $s + xi + yj + zk$ where $s, x, y, z \epsilon \mathbb{R}$ and $i^2 = j^2 = k^2 = ijk = -1$. They are similar to regular complex numbers, so it helps to have a review complex numbers before tackling quaternions. 

## Quaternion Operations

For all of these examples, I will be using two quaternions $q_a$ and $q_b$.

$$q_a = s_a + x_ai + y_aj + z_ak = [s_a, \mathbf{a}]$$

$$q_b = s_b + x_bi + y_bj + z_bk = [s_b, \mathbf{b}]$$

### Addition and Subtraction

Additition and subtraction for quaternions is defined element-wise. This means it is comparable to addition and subtraction for complex numbers because coefficients for the same complex value ($i$, $j$, or $k$) are combined.

Addition is defined as:

$$q_a + q_b = [s_a + s_b, \mathbf{a} + \mathbf{b}]$$

Subtraction is defined as:

$$q_a - q_b = [s_a - s_b, \mathbf{a} - \mathbf{b}]$$

### Multiplication

Quaternions can be multiplied just like complex numbers. Where complex numbers are following the rule $i^2 = -1$, quaternions follow $i^2 = j^2 = k^2 = ijk = -1$. This yields the following equations for simplifying terms in quaternions multiplication.

$$i = jk$$

$$j = ki$$

$$k = ij$$

These are important for understanding the derivation for multiplication. It is important to note that these are not commutative, and reversing the order will result in a term of the opposite sign.

$$=q_a q_b = [s_a, \mathbf{a}] [s_b, \mathbf{b}]$$

$$= (s_a + x_ai + y_aj + z_ak) (s_b + x_bi + y_bj + z_bk)$$

$$= s_as_b + s_ax_bi  + s_ay_bj  + s_az_bk  $$

$$ x_ais_b + x_aix_bi + x_aiy_bj + x_aiz_bk $$

$$ y_ajs_b + y_ajx_bi + y_ajy_bj + y_ajz_bk $$

$$ z_aks_b + z_akx_bi + z_aky_bj + z_akz_bk $$

This can be simplified and terms can be combined that share the same complex variable

$$= s_as_b - x_ax_b - y_ay_b - z_az_b + $$

$$ (s_ax_b + s_bx_a + y_az_b - z_ay_b)i + $$

$$ (s_ay_b + s_by_a + z_ax_b - x_az_b)j + $$

$$ (s_az_b + s_bz_a + x_ay_b - y_ax_b)k$$

This quaternion can also be written as a real number and vector in the imaginary space

$$= [s_as_b - x_ax_b - y_ay_b - z_az_b,$$

$$ s_a(x_bi + y_bj + z_bk) + s_b(x_ai + y_aj + z_ak) + $$

$$ (y_az_b -y_bz_a) + (x_bz_a - x_az_b) + (x_ay_b - x_by_a)] $$

This simplifies down 

$$ = [s_as_b + \mathbf{a} \cdot \mathbf{b}, s_a\mathbf{b} + s_b\mathbf{a} + \mathbf{a} \times \mathbf{b}]$$
