---
title: "Introduction"
date: 2018-04-10T23:03:23+02:00
draft: false
tags:
- python
- cloud foundry
- predix
- golang
- c++
- java
categories:
- post
description : "The first post of a long series"
---

# An Introduction

## Few words about myself
So I guess we need to start somewhere. So welcome to my website. I hope you will like it, and enjoy what I will put in it.
If you are loocking for coocking recipies this is probally not the best place :sweat_smile:.
I am software engineer, I am currently working at GE in France. I am what is called a **Software Architect** to me it means nothing else than old senior developer. I am not that old to be honest, I just happened to have writen quite a lot of code even though I spent a large share of my days in meetings, and trainings (I dispense them), Workshops (I run them). I am a **hardcore C++ programmer**, **lazy Java developer**,**opportunist python user**, and **enthusiastic gopher**. I do some **Js/Css/Html5** stuff sometimes but this really because, having a UI is always better to explain things. I haven't used Windows since ages, I code on a very expensive Mac (Thank you GE for that). My main focus since 2014 is cloud and distributed systems of all kind, with a sweet spot around computational intensive applications like (3D rendering, Image processing, ...). I have not worked nine years at Healthcare for nothing.

## A bit more words about what I do
I am doing a bit too much different things at the moment. I am more or less doing everything related to software pre-sales. I am reviewing commercial proposals, to make sure that what is about to be sent to the customer is actually doable on our cloud platform (sales … you know them), designing MVP to prove the value of digital transformation based on a simple use case. Present our Cloud Platform to customer curious to understand what we can do for their particular use case. Sometimes I am doing training, when customer wants to develop themselves on our platform. You may think that this leaves me very little time to write code. Actually, this is not the case, because I do not present architecture, that I do not know how to implement. So, I’ve done some Esp8266 and Esp32 low level C development to collect value from temperature and humidity sensors. Some Video processing on Raspberry Pi 3. At the end, all the data ends up in some sort of EventHub or Cloud Storage, then I run analytics in python/java and visualize them into an html5 web UI. Apart from that I am learning golang to test new distributed computational architecture.

## Enough talking show me some code
Ok so for this first article here a small piece of code I would like to comment.
{{< highlight cpp "linenos=table,hl_lines=8 15-17,linenostart=1:" >}}
#include "pistache/endpoint.h"

using namespace Pistache;

class HelloHandler : public Http::Handler {
public:
    HTTP_PROTOTYPE(HelloHandler)
    void onRequest(const Http::Request& request, Http::ResponseWriter response) {
        response.send(Http::Code::Ok, "Hello World");
    }
};

int main() {
    auto port = getenv("PORT");
    Pistache::Address addr(Pistache::Ipv4::any(), Pistache::Port(port != nullptr ? std::stoi(port) : 9080));
    auto opts = Pistache::Http::Endpoint::options()
        .threads(1);

    Http::Endpoint server(addr);
    server.init(opts);
    server.setHandler(Http::make_handler<HelloHandler>());
    server.serve();

    server.shutdown();
}
{{< / highlight >}}

This code is part of the **Pistache an elegant C++ REST framework** I found this project by luck and found it very interessting.
I do believe that writting cloud application in **C++** is the way to go for one main reasons: CPU/RAM => $. Since the more we consume the more we pay, language like golang or rust totally make sense. Go still have a GC, so it's sometime not enough for hardcore large scale services. Many Big Data Project like spark, flink, cassandra are doing some black magic to move data off the heap. So instead of doing some trick being able to manipulate data without doing extra stuff is a huge plus.

After viewing the code a simple makefile using cmake and we have generate the binary.
{{< highlight CMake "linenos=table,hl_lines=8 15-17,linenostart=1:" >}}
cmake_minimum_required(VERSION 2.8)

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})

project(pistache-demo)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Release")
elseif(CMAKE_BUILD_TYPE STREQUAL "Debug")
  set(CMAKE_VERBOSE_MAKEFILE ON)
endif(NOT CMAKE_BUILD_TYPE)

SET( CMAKE_VERBOSE_MAKEFILE TRUE )
add_definitions(-fPIC -std=c++17)
add_definitions(-Wall -Wextra)

SET( CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--no-as-needed -static-libstdc++ -static-libgcc" )

SET( PROJECT_SRC main.cpp )

add_executable(${PROJECT_NAME} ${PROJECT_SRC})
target_link_libraries(${PROJECT_NAME} pistache pthread)
{{< / highlight >}}

If you pay attention here I am linking statically against many things, I want to keep my dependencies to minimal for future cloud deployement.


## Running in the cloud

### Cloud Foundry

Deploying to cloud foundry is as usual very simple, and since we link most of our library in static, the only things we need it a to read the *PORT* env variable.

{{< highlight yaml "linenos=table,hl_lines=8 15-17,linenostart=1:" >}}
---
applications:
- name: pistache-demo
  random-route: yes
  buildpack: binary_buildpack
  memory: 32M
  disk_quota: 64M
  instances: 2
  path: .
  command: ./pistache-demo
{{< / highlight >}}

A cf push later and our application is up and running.

One things really important to notice is the ridicullous amount of memory consume by this simple dummy server 1.2M even if you put it under test with apache benchmark.
You get more or less the same, CPU does increase but latency is still the same.

### Docker

Super easy to deploy with a docker container, I am skiping the build process, this involves a multistage build more or less.

{{< highlight docker "linenos=table,hl_lines=8 15-17,linenostart=1:" >}}
FROM ubuntu:18.04

RUN useradd --create-home user -G sudo && \
    echo user":rootpw" | chpasswd
USER user
WORKDIR /home/user

EXPOSE 8080
ENV PORT 8080

COPY pistache-demo ./
CMD ["./pistache-demo"]
{{< / highlight >}}

I am droping privilege as soon as I can, running binary as root is not a good idea.
I am setting the env variable PORT to 8080 just to make things a bit more obvious.

{{< highlight bash "linenos=table,hl_lines=8 15-17,linenostart=1,nobackground=true" >}}
docker build -t blaurent/pistache-demo .
docker run -d blaurent/pistache-demo:latest
{{< / highlight >}}

And Boom done...

Super simple, who said C++ is hard and not meant for cloud services.
