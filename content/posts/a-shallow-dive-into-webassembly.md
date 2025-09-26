---
title: "A Shallow Dive Into Webassembly"
author: "Shriram Ravindranathan"
description: "Shoot yourself in the foot with C++, this time on the web"
date: 2022-03-18T19:22:00+05:30
draft: false
tags: [
    "c++",
    "webassembly",
    "intro",
    "webdev"
]
archives: ["2022", "2022-03"]
---
![Dive into WebAssembly](/images/WA.png)

If you've been following the buzz in the past few years, you must have certainly heard about [WebAssembly](https://webassembly.org), the new, low-level, general purpose programming language that can be used to compile code for the web.
Webassembly was created by Mozilla and Google to be [extremely fast](https://medium.com/@torch2424/webassembly-is-fast-a-real-world-benchmark-of-webassembly-vs-es6-d85a23f8e193), safe and to bring near native performance to code running in the browser.

Webassembly has been designed with performance in mind. It has been built on top of [LLVM](https://github.com/llvm/llvm-project) which means it will have access to all of LLVM’s optimizations and features.

Webassembly was created by the WebAssembly Community Group in 2015. The goal of Webassembly is to provide a compilation target that has a similar level of performance as native machine code but with the high-level convenience of JavaScript. It is supported by [all 4 of the major browser engines](https://webassembly.org/roadmap/) today.

## But Why WebAssembly?
With the near-native performance promised by WebAssembly, It is quite possible to [run really graphically intensive games](https://www.infoq.com/news/2019/07/doom3-web-assembly-port/) right in the browser. 
You surely must have used [Figma](https://www.figma.com) or atleast heard of it. It's a really powerful Graphic Design app which runs flawlessly, purely on the browser. The reason for its amazing performance is that It is powered by WebAssembly. 
Infact I am pretty confident that in the near future Audio/Video and Photo editing on the Web could go mainstream with such a leap in performance.


![WOW](/images/WOW.png)

## AssemblyScript

In order to write WebAssembly Code which runs in your browser, you don't actually need to learn WebAssembly syntax. WebAssembly is better thought of as a compilation target for other higher level languages which are easier to write code in.
In fact, if you know Typescript, you can jump right in to [AssemblyScript](https://www.assemblyscript.org/) which has [approximately 13000 npm downloads each week](https://www.npmjs.com/package/assemblyscript) at the time of this publication.

But if you're feeling really adventurous, you really gotta try compiling raw C++ into WebAssembly. Yes you heard me right, It is indeed possible to cause a [segfault in the browser](https://stackoverflow.com/questions/55583474/how-do-i-debug-an-emscripten-segmentation-fault-due-to-exceeding-the-available-d) with your code!

## Running C++ in the browser

With WebAssembly, it is possible to call a C++ function in the browser. You could use a toolchain like [Emscripten](https://emscripten.org) to do this but it is completely possible to compile C++ directly into WebAssembly's binary format `wasm` by setting `wasm32` as the target for clang++.

Let's write a function that does bitwise left-shifts and yields the nth power of 2

```c++
extern "C" {
  long long powerOf2(int n) {
    return (1 << n);
  }
}
``` 
C++ does [Name Mangling](https://en.wikipedia.org/wiki/Name_mangling) which changes the name of the exported function and we won't be able to call it from the browser if that happens. To suppress this behaviour, we need to wrap the function in `extern "C"`.

The C++ file can be compiled with the command :

```bash
clang++ --target=wasm32 --no-standard-libraries -Wl,--export-all -Wl,--no-entry -o power.wasm power.cpp
```

- `--target=wasm32` sets the target architecture to WebAssembly
- `--no-standard-libraries` because this target does not support libc++ (discussed later)
- ` -Wl,--export-all` exports all symbols (the function `powerOf2`) 
- `-Wl,--no-entry` tells the compiler that there is no `main` function

Keep in mind that you're going to need clang++ and thus need to have LLVM installed.
You could download the prebuilt packages depending on your OS or [build llvm from source](https://llvm.org/docs/GettingStarted.html)

once your wasm binary is generated, you can simply call that function in your webpage like so


```html
<!DOCTYPE html>

<html>
  <head>
    <meta charset="utf-8" />
    <title>WASM instantiateStreaming() test</title>
  </head>

  <body>
    <script>
      WebAssembly.instantiateStreaming(fetch("power.wasm")).then((obj) =>
        console.log(obj.instance.exports.powerOf2(62))
      );
    </script>
  </body>
</html>
``` 
[Code Based on source: WebAssembly JS-API documentation](https://github.com/mdn/webassembly-examples/blob/master/js-api-examples/instantiate-streaming.html)

Here, We are calling the `powerOf2` function with an argument of `62`

You can check the output in the console showing `1073741824n` and the [`n`
indicates](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt) that the number is infact a BigInt, which is right as we return a `long long` from the C++ function.

![Console Output](/images/console-wa-output.png)

As mentioned before, libc++ is not supported by the target `wasm32`, but in case you want to use it, you could target `wasi` [refer to this answer on stackoverflow](https://stackoverflow.com/a/67174440/11335534). 

And That's it! You've successfully run C++ in your browser. 
It's time now to port all of your legacy code over to the browser. Hope you could follow along with this. Happy memory leaks and race conditions!
