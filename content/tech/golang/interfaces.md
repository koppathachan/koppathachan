+++
title = "Using interfaces in golang to write testable code"
author = ["Akhil Sasidharan"]
lastmod = 2021-03-21T21:44:50+05:30
tags = ["programming", "golang", "interfaces", "tdd", "testing"]
draft = false
weight = 2001
+++

I like golang's interfaces. They're just a bunch of function signatures. Simple.
Any user-defined type can satisfy multiple interface types at once. Which is why
I could easily read a file and pass that to my function that takes an io.Reader
interface. \*os.File implements the Read(b []byte) (int, error).

I am a strong advocate of Test Driven Development or TDD. What I have
experienced is that the ['Eyes closed, head first, can't lose'](https://www.youtube.com/watch?v=WVIGAD5Kb70) approach to
writing code first and then thinking about testing it doesn't work. I end up
having to come up with creative ways to test my code. Writing mocks for my http
clients and maybe even setting up an actual db for my unit tests.

Interfaces makes it easier for me to think about my code before I actually
implement it. I have a set of interfaces I pass around that gets the job done.
For instance, I had to work on this feature where I had to send encrypted
request to a server to get a payment link. The encrypted request could be my
account information. Usually how this goes is, during the integration phase we
get a UAT environment with no encryption. Just to get the APIs tested and
working. Now I know, that I will need an encryption module (which needs to be
tested separately), and I need a way to turn it off for other developers working
on integrating the APIs.

My interfaces looks like this:

<a id="code-snippet--Eg1"></a>
{{< highlight go "linenos=table, linenostart=1" >}}
// The interface that I defined which will take the account info and
// return the payment link.
type Payment interface {
	GetLink(a AccountInfo) (*Link, error)
}

// This one will return an io.Reader implementation, io.Reader to be
// passed to the Post(in io.Reader) function. See the Service interface
type RequestBuilder interface {
	Marshal(a AccountInfo) (io.Reader, error)
}

// This one will read the response from the Post(in io.Reader) and return
// a reference to the Link object.
type ResponseBuilder interface {
	// let's not get into what Response actually is
	Unmarshal(r Response) (*Link, error)
}

// Service is nothing but a regular http method wrapper.
type Service interface {
	Post(in io.Reader) (*Response, error)
}
{{< /highlight >}}

The code that ties all things together looks like this.

<a id="code-snippet--Eg2"></a>
{{< highlight go "linenos=table, linenostart=1" >}}

type pay struct {
	rqb RequestBuilder
	rsb ResponseBuilder
	s   Service
}

// ta-da!
func New(rqb RequestBuilder, rsb ResponseBuilder, svc Service) Pay {
	return pay{
		rqb: rqb,
		rsb: rsb,
		s:   svc,
	}
}

// Wiring it all up.
func (p pay) GetLink(a AccountInfo) (*Link, error) {
	// we marshall the account info
	in, err := p.rqb.Marshal(a)
	if err != nil {
		return nil, err
	}
	// we post the data
	r, err := p.s.Post(in)
	if err != nil {
		return nil, err
	}
	// and we unmarshal the response to return the link
	return p.rsb.Unmarshal(*r)
}
{{< /highlight >}}

Assuming I have a package 'paycompany' that implements pay.RequestBuilder,
pay.ResponseBuilder and pay.Service interaces for the company (paycompany) I am
integrating with I can now do the following.

<a id="code-snippet--Eg3"></a>
{{< highlight go "linenos=table, linenostart=1" >}}
// without encryption
linkSvc = pay.New(
	paycompany.NewRequestBuilder(),
	paycompany.NewResponseBuilder(),
	paycompany.NewService(),
)
{{< /highlight >}}

or

<a id="code-snippet--Eg4"></a>
{{< highlight go "linenos=table, linenostart=1" >}}
//  with encryption
linkSvc = pay.New(
	paycompany.NewEncRequestBuilder(),
	paycompany.NewEncResponseBuilder(),
	paycompany.NewService(),
)
{{< /highlight >}}

and eventually just call

<a id="code-snippet--Eg5"></a>
{{< highlight go "linenos=table, linenostart=1" >}}
l, err := linkSvc.GetLink(a)
// l  has the link
{{< /highlight >}}

Now if I want to test the Pay interface I can do that too. I just have to mock
the **pay.Service** interface (we don't want to call the actual service). Now I
can test the parts and the whole.

I also created an interface for encryption that can be tested
separately. In my case I am using a public and private key based encryption.

<a id="code-snippet--Eg4"></a>
{{< highlight go "linenos=table, linenostart=1" >}}
type Encrypter interface {
	Key() string
	Encrypt(p string) string
	Hash(p string) string
	KeyEncrypt(k string) string
}

// The response if also encrypted of course.
type Decrypter interface {
	Decrypt(p string) string
}
{{< /highlight >}}
