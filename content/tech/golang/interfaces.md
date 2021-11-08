+++
title = "Using interfaces in golang to write testable code"
author = ["Akhil Sasidharan"]
lastmod = 2021-11-08T22:37:59+05:30
tags = ["interfaces", "tdd", "testing"]
categories = ["programming", "go"]
draft = true
+++

I like golang's interfaces. They're just a bunch of function signatures. Simple.
Any user-defined type can satisfy multiple interface types at once. Which is why
I could easily read a file and pass that to my function that takes an io.Reader
interface. \*os.File implements the Read(b []byte) (int, error).

<a id="code-snippet--io.Reader"></a>
{{< highlight go "linenos=table, linenostart=1" >}}
// the io.Reader interface
type Reader interface {
	Read(p []byte) (int, error)
}
{{< /highlight >}}

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
package pay

// custom reader method that takes other methods
type Reader interface {
	// add other methods you want
	io.Reader
}

// The interface that I defined which will take the account info and
// return the payment link.
type Payment interface {
	GetLink(a Reader) (*Link, error)
}

type AccountInfo struct {
	// account info goes here
}

// We define the Read method on AccountInfo so that it satisfies
// the io.Reader interface
func (a AccountInfo) Read(p []byte) (int, error) {
	// marshal it and ready for transport
	bytes, err := json.Marshal(a)
	if err != nil {
		return 0, err
	}
	// io.EOF is a special error that denotes no more bytes to read
	// copy returns the number of bytes copied
	return copy(p, bytes), io.EOF
}

// Service is nothing but a regular http method wrapper.
// with some custom messages to extract Link from Response.
type Service interface {
	Post(in Reader) (*Response, error)
	Link(r Response) (*Link, error)
}
{{< /highlight >}}

Now type **pay.AccountInfo** implements the **io.Reader** interface. It is clear
that there is no encryption involved here. For encryption I define a separate
struct.

The code that ties all things together looks like this.

<a id="code-snippet--Eg2"></a>
{{< highlight go "linenos=table, linenostart=1" >}}
package pay

type pay struct {
	s Service
}

// ta-da!
func New(svc Service) Pay {
	return pay{
		// this can have multiple
		// implementations depending on the
		// service API(testing and production)
		s: svc,
	}
}

// Wiring it all up.
func (p pay) GetLink(a Reader) (*Link, error) {
	// a satisfies the io.Reader interface
	r, err := p.s.Post(a)
	if err != nil {
		return nil, err
	}
	return p.s.Link(*r)
}
{{< /highlight >}}

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

I have to use the above encrypter package (assume it is implemented) to
encrypt the AccountInfo type so that I can use it in our production systems
pointing to the actual service API. In my package 'paycompany' that
implements the **pay.Service** interface for the company, 'paycompany', I am
integrating with I can now do the following.

<a id="code-snippet--Eg3"></a>
{{< highlight go "linenos=table, linenostart=1" >}}
// without encryption
package paycompany

func NewService() pay.Service {
	// implement here
}

// the format to send the encrypted data in
type EncryptedData struct {
	Hash string
	Body string
	Key  string
	Iv   string
}

// inherits the AccountInfo type
type EncAccountInfo struct {
	pay.AccountInfo // struct embedding.
}

// implements the io.Reader interface and also encrypts the bytes
func (e EncAccountInfo) Read(p []byte) (int, error) {
	bytes, err := json.Marshal(e)
	if err != nil {
		return 0, err
	}
	var bodystring = string(bytes)
	// get iv somehow
	iv := "auniquevalue"
	// get the encrypter somehow
	var enc Encrypter = crypt.NewEncrypter(iv)
	ed := EncryptedData{
		Hash: enc.Hash(bodystring),
		Body: enc.Encrypt(bodystring),
		Key:  enc.Encrypt(enc.Key()),
		Iv:   iv,
	}
	ebytes, err := json.Marshal(ed)
	if err != nil {
		return 0, err
	}
	return copy(p, ebytes), io.EOF
}
{{< /highlight >}}

Et voilà, we have the same Read method on the same (kind of) type but now it is
encrypted. I just overrode the Read method. To use it just create a new type from
the original object and call the same method as shown below.

<a id="code-snippet--Eg5"></a>
{{< highlight go "linenos=table, linenostart=1" >}}

var linkSvc pay.Payment = pay.New(
	paycompany.NewService(),
)

var accInfo pay.AccountInfo // once you get this somehow
var encAccInfo = paycompany.EncAccountInfo{accInfo}

// calls the pay.AccountInfo.Read() method underneath
l, err := linkSvc.GetLink(accInfo)

// calls the paycompany.EncAccountInfo.Read() method underneath
l, err = linkSvc.GetLink(encAccInfo)
// l  has the link
{{< /highlight >}}

If I want to test the Payment interface I can do that too. I just have to mock
the **pay.Service** interface (we don't want to call the actual service). Now I
can test the parts and the whole. Infact I only need to test the
pay.Service.Link() function and my Read methods in all the types.

I have left out the decryption part and assumed that the
**paycompany.NewService()** can handle both the testing and production
environments.

Happy Go coding.
