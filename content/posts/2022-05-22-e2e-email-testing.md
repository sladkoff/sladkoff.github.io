---
title: E2E email testing with temp-mail
date: "2022-05-25"
comments: true
buyMeACoffee: true
draft: false
toc: true
Cover: 
tags:
  - testing
  - email

---

## The Problem

You have a web app that sends out emails using a service like SendGrid or similar when a user signs up. It's actually not important how the mail delivery is implemented. You want to test
that an email _is_ delivered at all using an _automated_ test runner like Cypress.

This is what the sign-up implementation could look like.

![diagram](/img/generated/e2e-email-sequence.svg)

Note that we don't want to reconfigure the app under test to use a **different** mail API or SDK (like [MailTrap]()) to reroute our whole mailing traffic via a different service. We don't do that because that essentially prevents us from truly testing the app end-to-end.

## The solution

Let's first have a look at our high level test spec.

1. **Given** a new email address and password
2. **When** I sign up to my app
3. **Then** I want to receive an email

In Step 2 we will need to provide a valid email
for our app. In step 3 we will need to check that an email arrived so we need some sort of mailbox. Let's define some requirements for our mailbox.

1. The mailbox should give us **unique** addresses to use per test.
2. The mailbox should require **minimal manual configuration**.
3. The mailbox should be accessible via an **API**.
4. The mailbox should be **cheap af**.

**[1secmail.com](https://1secmail.com) to the rescue!**

The web version looks quite hideous but it has a really simple [API](https://www.1secmail.com/api/).

It basically functions as a catch-all email mailbox. So you don't need to _create_ temp addresses.
You can _use_ any temp address in your tests (e.g. during sign-up). 
You can then use the temp address as _key_ to fetch email for the temp address.

So our automated test looks like this under the hood.

- Generate a temp email address like `test-user-${unique-suffix}@1secmail.com` (no API call necessary)
- Use the generated temp email address in the test to sign up to the app under test
- Check for mails using the 1secmail API

Here's an example axios implementation of the API call:

```typescript
const suffix = faker.random.numeric(8)

type Message = {
  id: string
  from: string
  subject: string
  date: string
}

async getMessages(
  login: string,
  domain: string
): Promise<Message[]> {
  const response = await axios.get<Message[]>(
    'https://www.1secmail.com/api/v1/',
    {
      params: {
        login: `test-user-${suffix}`,
        domain: '1secmail.com',
      },
    }
  )
  return response?.data
}
```

Let's double check if this approach fulfils our defined requirements.

- [x] We get **unique** temporary addresses per test.
- [x] The mailbox is catch-all so **no configuration** is necessary.
- [x] There's a **simple API**.
- [x] It's **free**.

Note: the public nature of this (anyone can read any temp mailbox) makes this unsuitable for tests where some sort of sensitive data is exchanged. 

### Why not aliases?

You could create an email address at a public email provider like Protonmail. 

You could use unique aliases for each test like `test-user+timestamp@protonmail.com`.

I know they do have some sort of public API for you to use in your automated tests.

You can get probably get away with this for quite some time. My previous setup involved
a prepared email address with aliases at Protonmail. Unfortunately they terminated my account
due to a violation of their TOS - apparently you're not allowed to use automated software
to spam their system. Who would have thought.

### Why not MailTrap?

MailTrap looks promising because it's supposed to be an email testing service but it's not
suited for real E2E tests on production if you're not willing to pay premium cash. 
You don't get any public email address up until the Business plan which costs 50$ / month!

In other words, the lower tier plans only allow you to include MailTrap's SDK into your app
or use SMTP to send mail directly to MailTrap. This is not E2E testing because you're modifying
the app's configuration. 

Basically it's useless and I was disappointed enough to write them a mail when I found out about this...
