---
title: Building REST services with Spring

categories:
- Spring Tutorial

date: 2020-04-29
---
REST has quickly become the de-facto (事实上) standard for building web services on the web because they're easy to build and easy to consume (消费).

There're much larger discussion to be had about how REST fits in the world of microservices, but for this tutorial - let's just look at building RESTful services.

Why REST? REST embraces (拥抱) the precepts (方案) of the web, including its architecture (架构), benefits (优势), and everything else.  This is no surpise given its author, Roy Fielding, was involved (参与) in probably a dozen specs which govern (统治) how the web operates.

What benefits? The web and its core protocol, HTTP, provide a stack of features:
- Stiable (合适的) action (GET, POST, PUT, DELETE, ...)
- Caching
- Redirection and forwarding
- Security (encryption 加密 and authentication)

There are all critical factors (要害因素) on building resilient (有适应力的) services. But that is not all. The web is built out of lots of tiny specs, hence (因此) it's been able to evolve easily, without getting bogged down in "standards wars".

Developers are able to draw upon (凭借) 3rd party toolkits that implement these diverse (不同的) sepcs and instantly (立刻) have both client and server technology at their fingertips (指尖).

So  building on top of HTTP, REST APIs provide the means to build flexible APIs that can:
- Support backward compatibility (兼容)
- Evolvable (可展开的) APIs
- Scaleable services
- Securable services
- A spectrum (系列) of stateless (无状态) to stateful (有状态) servcie

What's important to realize is that REST, however ubiquitous (似乎无所不在的), is not a standard, per se (本质上), but an approach, a style, a set of constraints (约束) on your architecture that can help you build web-scale systems. In this tutorial we will use the Spring protfolio (作品集) to build a RESTful service while leveraging (对…施加影响) the stackless features of REST.

## Getting Started
As we work through this tutorial, we'll use Spring Boot. Go to Spring Initializr and select the follow:
- Web
- JPA
- H2
- Lombok

Then choose "Generate Project". A `.zip` will downloaded. Unzip it. Inside you'll find a simple, Maven-based project including a `pom.xml` build file (NOTE: You can use Gradle. The example in this tutorial will be Maven based.)

Spring Boot can work with any IDE. You can use Eclipse, IntelliJ IDEA, Netbeans, etc. The Spring Tool suite is an open-source, Eclipse-based IDE distribution (发行) that provides a superset of the Java EE distribution of Eclipse. It includes features that making working with Spring application even easier. It is, by no means, required. But consider it if you want that extra oomph (特质) for your keystrokes (按键). Here's a 