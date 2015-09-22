---
layout: post
title: "Building GoPhish - How to Send Email with Go"
date: 2014-01-03 22:06
comments: true
categories:
- gophish
- go
---
<img src="{{root_url}}/images/headers/go_email.png"/>
### Introduction
I've been playing around with Go for about a month now, and I've *really* grown to like it. After getting used to the syntax and remembering what a pointer is for (thanks, Python), Go has become a favorite language to develop with. I'm even using it for the [Matasano Crypto Challenges](http://www.matasano.com/articles/crypto-challenges/) (which are *awesome*).

While the standard library in Go is definitely robust, being a young language, there are a few niceties that aren't yet included. Sending email is one of them. Don't get me wrong, Go has a wonderful [SMTP](http://golang.org/pkg/net/smtp/) package, [MIME](http://golang.org/pkg/mime/) package, and even a [Mail](http://golang.org/pkg/net/mail/) package (which *only* parses existing email messages). However, there is no library to actually **create** emails in a meaningful way. Since [Gophish](https://github.com/jordan-wright/gophish) relies heavily on sending emails, I've sought to change this. And, after reading more RFC's than I normally prefer, I believe I've created a package that provides intuitive, robust, and flexible email creation and sending called [email](https://github.com/jordan-wright/email).

Let's see how to use it.
<!--more-->
### How You *Normally* Send an Email in Go
To send email in Go, one needs to:

*  Create a byte slice of the email message (conforming to all needed RFC's)
*  Send this email using the SMTP library

The second part is easy, the first part is not. Sure, sending a simple text message may be straight forward, but things get tricky when you want to cover things like supporting HTML and text body types, attaching files, supporting CC and BCC recipients, etc.

### Sending Email Using the ```email``` Package
To make this easier, I have created the [```email```](https://github.com/jordan-wright/email) package (full documentation [here](http://godoc.org/github.com/jordan-wright/email)). This package allows users to create emails with a variety of options, and send them easily. Examples say more than I can, so here is some code showing how to use the package.

#### Installing the Package
```
go get github.com/jordan-wright/email
```

#### Creating a New Email
``` go
package main

import "github.com/jordan-wright/email"

func main() {
	e := email.NewEmail()
}
```
#### Setting the Subject, To, From, Bcc, Cc
``` go
e.Subject = "Awesome Subject"
e.From = "Jordan Wright <test@gmail.com>"
e.To = []string{"test@example.com"}
e.Bcc = []string{"test_bcc@example.com"}
e.Cc = []string{"test_cc@example.com"}
```
#### Setting the Content (HTML & Text)
``` go
e.Text = "Text Body is, of course, supported!"
e.HTML = "<h1>Fancy Html is supported, too!</h1>"
```
#### Attaching a File
``` go
e.AttachFile("test.txt")
```
You can also use the [```Attach```](http://godoc.org/github.com/jordan-wright/email#Email.Attach) function to attach content directly from an io.Reader.
#### Sending the Email (Using Gmail as Example)
``` go
e.Send("smtp.gmail.com:587", smtp.PlainAuth("", "test@gmail.com", "password123", "smtp.gmail.com"))
```
#### Conclusion
I hope this package will be useful to those that need to send email from their Go projects. I am excited to continue working on Gophish - you can expect a big update soon! Until then, as always, feel free to leave any questions or comments below.

-Jordan ([@jw_sec](http://twitter.com/jw_sec))
