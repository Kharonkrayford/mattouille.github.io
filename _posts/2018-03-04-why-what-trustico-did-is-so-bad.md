---
title: "Why what Trustico did is so bad"
layout: post
tags:
- Security
category: [Security]
---

If you haven't heard, last Wednesday it was revealed that Trustico, a large reseller of TLS certificates, violated customer trust by emailing another reseller 23,000 of their customer private keys. That's really the gist of what they did, what these articles don't really cover is the underlying reasons why this is directly bad and why what it means is even worse.

There's this really sweet document called the ("Baseline Requirements for the Issuance/Management of Publicly-Trusted Certificates")[https://cabforum.org/wp-content/uploads/CA-Browser-Forum-BR-1.5.4.pdf]. In that document there's section 6.1.2 which states:

> Parties	other	than	the	Subscriber	SHALL	NOT	archive	the	Subscriber	Private	Key	without	authorization	by	the	
Subscriber.	
> If	the	CA	or	any	of	its	designated	RAs	generated	the	Private	Key	on	behalf	of	the	Subscriber,	then	the	CA	
SHALL	encrypt	the	Private	Key	for	transport	to	the	Subscriber.	
> If	the	CA	or	any	of	its	designated	RAs	become	aware	that	a	Subscriber’s	Private	Key	has	been	communicated	
to	an	unauthorized	person	or	an	organization	not	affiliated	with	the	Subscriber,	then	the	CA	SHALL	revoke	all	
certificates	that	include	the	Public	Key	corresponding	to	the	communicated	Private	Key.

This helps spell out and dispel some things. CA's are allowed to generate keys on behalf of the customer, however, they are not allowed to archive said keys for any reason. If the CA generates the key on behalf of the customer, they must encrypt said key for transmission to the customer.

## What I generally regard as safe

I don't let a CA generate private keys for me. Private keys are used for encryption, but what they really are is a method of identity. There's three important pieces to secure TLS communication: the private key, the public key, and a CSR (Certificate Signing Request). These three things form a trust relationship between a well known Certificate Authority and something on the internet. In laymens terms the CA is trusted to have a root private key and to use that to sign off on relationships with endpoints on the internet given some burden of proof.

The public, private keys, and CSR should be generated on whatever network you intend to assume trust for. The real point here is that the private key never leaves.

Next the CSR is transmitted to the CA. The CSR contains some metadata and the public key encoded in ASN.1. The CA verifies any information they feel they need to verify to establish trust, sign the certificate, and transmit it back to you. The CA should never actually know your private key, they **only** *establish trust*.

## Failures

TLS issuance, and largely cryptography in general, was made from the ground up to be automated. There's a couple major failures:

1. A CEO thought it was a good idea to transmit private keys and CSR's (which you now know contain public keys, am I right?) across email. Whether it was encrypted or not, this action violates the trust of every customer they've had. CA's jobs are to establish trust, and the reason they're given the *trust to do that* is because people trust their processes.

2. Private keys were stored in the first place. We know from our Baseline Requirements documents that this isn't acceptable under any terms.

## Action Items

1. Transmitting private keys is dangerous. The Baseline Requirements should, in my opinion, be updated to get rid of CA generated private keys.

2. The CEO of Trustico does not understand the underlying technology or premises of the company he runs. He should be terminated immediately.

3. Each major CA should be evaluated by a third party to assess the CA's adherence to the Baseline Requirements and CA's outside of the Baseline Requirements should be forced to notify current and future customers of this without explanation. I say without explanation, because Trustico said they needed to maintain private keys in order to revoke them. I'm not sure what world they live in given this statement.

That's all my ramblings for now, hopefully it's helped some people, thanks for reading!
