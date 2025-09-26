---
title: "C++ for the Web with Drogon"
author: "Shriram Ravindranathan"
description: "Create a basic REST API with C++ and Drogon"
date: 2023-06-30T21:23:07+05:30
archives: ["2023", "2023-06"]
draft: false
tags: [
    "c++",
    "intro",
    "webdev"
]
---
[Drogon](https://drogon.org) is currently [the fastest web server in the world](https://www.techempower.com/benchmarks/#section=data-r21). If you already know a bit of C++, it wouldn't hurt to try it out. I personally think drogon is great, however I do find its documentation somewhat lacking which is all the more reason to contribute to them. If you find that drogon is right for you, please consider [contributing to / sponsoring their project](https://github.com/drogonframework/drogon).

In this blog, we'll see how we can set up a basic REST API using drogon and CMake.

The code snippets I've provided apply to macOS and Linux. However, you can still follow along if you're on windows and have worked with CMake and VS before.

## Project Setup

If you're like me and you don't mind using CMake with submodules, we can set up drogon like so:

Get the pre-requisite libraries installed in accordance with [the official documentation](https://github.com/drogonframework/drogon/wiki/ENG-02-Installation). For macOS, I had to install `ossp-uuid` and `jsoncpp`.

Create a CMakeLists.txt in your project root.
```sh
touch CMakeLists.txt
```
Initialize Git in your project root.
```sh
git init
```
Add drogon as a submodule

```sh
git submodule add https://github.com/drogonframework/drogon.git external/drogon
```
Pull drogon's submodules (trantor)
```sh
git submodule update --init --recursive
```

At this point you've got your dependencies setup. Let's go ahead and add some code for our basic REST API.

Create a new directory for our source files

```sh
mkdir src
```

And add our entrypoint 

```sh
touch src/main.cpp
```

Here's a basic `CMakeLists.txt` file that will build our project 

```cmake
cmake_minimum_required(VERSION 3.25.2)

project(
    drogontutorial
    VERSION 0.1
    DESCRIPTION "Drogon CMake Tutorial"
    LANGUAGES CXX
)

# compiler flags
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")

add_executable(${PROJECT_NAME} src/main.cpp)

add_subdirectory(external/drogon)

target_link_libraries(${PROJECT_NAME} PRIVATE drogon)
```

And this will be our `src/main.cpp` 

```c++
#include <drogon/drogon.h>
int main()
{
    using namespace drogon;
    app().setLogPath("./")
        .setLogLevel(trantor::Logger::kWarn)
        .addListener("0.0.0.0", 8080)
        .setThreadNum(8)
        .registerHandler("/hello?name={name}",
            [](const HttpRequestPtr& req,
                std::function<void(const HttpResponsePtr&)>&& callback,
                const std::string& name)
            {
                Json::Value json;
                json["result"] = "ok";
                json["message"] = std::string("hello,") + name;
                auto resp = HttpResponse::newHttpJsonResponse(json);
                callback(resp);
            },
            { Get,"LoginFilter" })
        .run();
}
```

## Building and Running the App

So far your directory structure should look something like this:

```bash
.
├── CMakeLists.txt
├── external
│   └── drogon
└── src
    └── main.cpp

4 directories, 2 files
```
Go ahead and create a directory to house our build files

```sh
mkdir build && cd build
```
Now lets generate the Makefile using 

```sh
cmake ..
```

and now to compile our application,
```sh
make -j$(nproc)
```

Hopefully after all this, you should find an executable called `drogontutorial` in the build directory.

let's run this file to start our server.

```sh
./drogontutorial
```

Now, your server should be listening on port 8080. Test it out by sending your name to `/hello`

```sh
curl http://localhost:8080/hello?name=yourname
```

And just like that you've got a starter project to write your own REST APIs in C++.

If you find this nice, do check out their [examples on github](https://github.com/drogonframework/drogon/tree/master/examples).

Until next time!

