---
title: Flipping an image
excerpt: Flip images horizontally, vertically or both
order: 100
---

Flips the image horizontally, vertically or both. Valid values are `x`, `y` and `xy`. The default value is `0`, which means it is not set.

> The `flip` parameter can be added to the querystring as either `fl` or `flip`

## Flip horizontally

`https://cdn.example.com/images/dog.jpg?flip=x`

![Dog flipped on the X axis](/assets/cdn/dog-w600-flip-x.jpeg "Image credit: Yamon Figurs (https://unsplash.com/@yamonf16)")

## Flip vertically

`https://cdn.example.com/images/dog.jpg?flip=y`

![Dog flipped on the Y axis](/assets/cdn/dog-w600-flip-y.jpeg "Image credit: Yamon Figurs (https://unsplash.com/@yamonf16)")

## Flip horizontally and vertically

`https://cdn.example.com/images/dog.jpg?flip=xy`

![Dog flipped on both axes](/assets/cdn/dog-w600-flip-xy.jpeg "Image credit: Yamon Figurs (https://unsplash.com/@yamonf16)")