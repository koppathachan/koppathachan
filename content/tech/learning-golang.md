+++
title = "Go. Weird And Awesome"
author = ["Akhil Sasidharan"]
date = 2020-05-16T21:23:40+05:30
lastmod = 2021-11-08T22:37:40+05:30
tags = ["golang", "go", "programmingbasics"]
categories = ["programming", "go"]
draft = false
+++

November 10th, 2019 marked the 10th anniversary of this awesome programming language called Go. The Go community in Bangalore organized a meetup (which was also their 50th meetup) hosted by SAP Labs. The meetup introduced me to gRPC and writing custom plugins for gRPC. The meetup also introduced me to how the Go runtime scheduler works and how it has achieved high performance concurrency. I was just a beginner in Go and starting to fall in love with the language, and knowing the genesis story of Go helped me understand some of the weirdness I felt about the language initially.
After starting with some basic programming in Go (creating a basic http server and some middleware), the language grew on me and quickly became my favourite programming language. The designers of Go set out to create a fast, productive, and fun programming language, and that's exactly what they did. Let's take a brief tour through some of my learning and experiences.
I have taken less time to learn Go than any other programming language. It has ~25 keywords, but it can do so much. It is garbage collected, compiled, statically typed, and has a small build time. It compiles to machine code, so it performs like C.
Many languages avoid enforcing semicolons at the end of a statement. Go, following in the path of C, does enforce semicolons to terminate statements but doesn't bother developers with it. They insert one when Go code is compiled and they have defined rules for it. You can add it yourself too, but idiomatic Go avoids it. One consequence of this decision is that braces for control structures (if, for, switch, and select) should be on the same line and not on a new line, which adds to the readability of Go. Programmers do not have to make rules instructing how and where braces are used.
I remember learning to code (in C++), and missing to add break statements in my switch cases and thinking, "that's not intuitive". I think, as a beginner to coding, those were the most irritating logical errors I have had to debug. The fall through behavior that is common in all programming languages (except python, which does not have a switch case) is useful sometimes. However, since it is not intuitive, I guess every programmer would have made the mistake of missing that break statement. Go switch cases do not fall through unless you want to (using the "fallthrough" keyword) and I like that.

<a id="code-snippet--Switch"></a>
{{< highlight go "linenos=table, linenostart=1" >}}
package main

import (
   "fmt"
)

func main() {
   fmt.Print("Go runs on ")
   switch 2 {
   case 1:
	fmt.Println("1")
	fallthrough
   case 2:
        fmt.Println("2")
	fallthrough
   case 3:
	fmt.Println("3")
   }
}
{{< /highlight >}}

Go also renames the while loop. Like the Go tour specifies "C's while is spelled for in Go." One less keyword to remember.
Go's concurrency model involves using Goroutines. Goroutines can be thought of as lightweight threads. They take very little stack space and the stack can grow and shrink according to program requirements. A program may just have one OS thread but a thousand Goroutines. Moreover, if one of the Goroutines is blocking, then the Go runtime moves all the other Goroutines into a newly spawned OS thread. All of this is done automatically by the Go runtime. Goroutines communicate using channels, which are special data structures. Channels can be buffered and Goroutines can push and pull data from it.

<a id="code-snippet--While"></a>
{{< highlight go "linenos=table, linenostart=1" >}}
package main

import "fmt"

func main() {

    messages := make(chan string)

    go func() { messages <- "ping" }()

    msg := <-messages
    fmt.Println(msg)
}
{{< /highlight >}}

Go is not intuitively Object Oriented. One of the main reasons for this is because it does not support inheritance. This is not by accident, but an intentional design choice. Go considers inheritance of the classical style fragile. Instead, it uses Composition. In my opinion, Go is responsibly Object Oriented. I have often noticed that, programmers write code, using languages like Java and C# where there is only one way (OO way), but still don't achieve OO programs. With Go, you can have methods on any type, not just structures. Go interfaces are just a set of methods and you can compose an interface with multiple types that satisfy these interfaces and any other type that satisfies the whole interface.

<a id="code-snippet--Object Oriented"></a>
{{< highlight go "linenos=table, linenostart=1" >}}
package main

import (
  "fmt"
)

type author struct {
  firstName string
  lastName  string
  bio       string
}

//a method on the struct author
func (a author) fullName() string {
  return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}

//author is embedded in struct post
type post struct {
  title     string
  content   string
  author
}

//details is attached on post
func (p post) details() {
  fmt.Println("Title: ", p.title)
  fmt.Println("Content: ", p.content)
  //can be accessed directly
  fmt.Println("Author: ", p.fullName())
  //can be accessed with the struct name also.
  fmt.Println("Bio: ", p.author.bio)
}
{{< /highlight >}}

Go supports encapsulation at the package level. Exported methods in Go start with a capital letter, and that is one of the things that makes Go so readable. This is perhaps what I like most about Go syntax. Like other languages, we are not burdened with the decision of choosing between camel cases or pascal cases. Go makes it for us.

<a id="code-snippet--Encapsulation"></a>
{{< highlight go "linenos=table, linenostart=1" >}}
package math

//add function is public.
//access it outside the package like so:
// math.Add(4,5)
func Add(a int, b int) int {
	return a + b;
}

//not accessible outside the package math.
func addPrint(a int, b int) {
	fmt.Println(Add(a,b))
}
{{< /highlight >}}

At work we decided, after much deliberation and debate, to use Go for one of our data processing microservices (Go concurrency model and performance benchmarks won the debate). We were designing a live data collection agent(for Managed Print Services) which consumed a huge amount of JSON data(about printers and their print count, cost etc.) from multiple paged REST APIs, categorised them based on various parameters, stored them and presented it. When we did some googling comparing Apache Spark and a custom solution in Go, we found that even though development effort for spark was lesser, the Go solution would perform better, was simpler and more efficient. The first thing that struck me was that Go is just a programming language(spark is a framework) and it was still easy to build data processing pipelines with just language constructs.
I have come to describe OO Go as "Responsible Object Oriented Programming", because it has truly bettered the way I write OO code. When it comes to concurrency, the amazing Goroutines has made me responsible as well as fearless in my coding. I would make Go my top choice just for its general purposiveness, small learning curve, and its refreshing take on object oriented programming.
