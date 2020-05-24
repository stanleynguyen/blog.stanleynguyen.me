+++
title = "is-thirteen in Go"
date = 2020-05-01T14:53:23.322Z
author = "stanleynguyen"
keywords = ["go", "javascript", "is-thirteen", "funny", "open-source", "thirteen"]
cover = "https://dev-to-uploads.s3.amazonaws.com/i/d5oskx7733k0xfjs8r0l.png"
summary = "A Golang module to check if a number or a string is thirteen"
+++

As COVID-19 lockdown is still going on, Labor Day is another indoor day just like how it has been for the last month. I'm surprised I even remember what day it is. To make myself feel less useless and bored, I decided to write a useless (?) Go module port called [is-thirteen](https://github.com/stanleynguyen/is-thirteen) from [its original JS version](https://github.com/jezen/is-thirteen) . With its extensive API, you can:

### Check whether a number is 13

```go
...
	is.Number(13).Thirteen()               // true
	is.Number(12.8).Roughly.Thirteen()     // true
	is.Number(6).Within(10).Of.Thirteen()  // true
	is.Number(2007).YearOfBirth.Thirteen() // true

	// check your math skillz
	is.Number(4).Plus(5).Thirteen()     // false
	is.Number(12).Plus(1).Thirteen()    // true
	is.Number(4).Minus(12).Thirteen()   // false
	is.Number(14).Minus(1).Thirteen()   // true
	is.Number(1).Times(8).Thirteen()    // false
	is.Number(26).Divides(2).Thirteen() // true
...
```

or

### Check whether a string is 13

```go
...
	// check your spelling and chemistry skillz
	is.String("tHirTeEn").Thirteen()              // true
	is.String("nethtire").AnagramOf.Thirteen()    // true
	is.String("neetriht").Backwards.Thirteen()    // true
	is.String("aLumInUm").AtomicNumber.Thirteen() // true
...
```

[is-thirteen](https://github.com/stanleynguyen/is-thirteen) is stable with 98% test coverage. Check it out over on [Github](https://github.com/stanleynguyen/is-thirteen)!
