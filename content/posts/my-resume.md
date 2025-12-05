---
title: "My Resume"
date: 2025-12-05T14:53:43+01:00
draft: false
---

More than a resume, this is a manifesto of what I aim to be as a developer.

### As a kid

Born in 1986, I spent most of my childhood reading books and trying to understand the world.
I took a fascination for **languages**, the **rubik's cube**, and other systems.

One of my biggest question was and remains:

> *How on earth does the world work?*

### As an adult

I have been:

- a **sheet metal worker** (2007-2010), in France and Germany.
  Trained as a technical designer and worked as a craftsman in various industries (notably shipbuilding).
- a **student of psychology** (2011-2015), in the university of Caen (Normandy).
  I started to specialize as a clinical psychologist in a master's degree,
  aiming to become a therapist, but I quit after realizing I would make a poor psychologist
  due to my own unresolved issues.
- a **german teacher** in french middle & high schools (2015-2018).
  I love german and I shine in front of a public.

### Becoming a developer

Fascinated by free software and open protocols,
I went on a quest to learn programming.
Advised by the awesome [Jean-Philippe Cugnet](https://github.com/ejpcmac),
I learned Rust, and by extension, programming, with the [Rust book](https://doc.rust-lang.org/stable/book/) in 2019.

I was a linux user already, and learned the ropes of system administration by hosting a minecraft server
on a PC inherited from my dad, to play on with friends during the pandemic.

I met with excellent developers in Caen, Normandy, who taught me what a REST-full API is,
and how to document it with [OpenAPI specification](https://en.wikipedia.org/wiki/OpenAPI_Specification).

Yet I was plagued with a riddle: **how to deploy apps on the internet?**
And I kept hearing about a company called Clever Cloud.

Searching for a job in 2020 was hard since I was kinda old for a junior developer,
but it turns out Clever Cloud was searching for a Rust-capable apprentice. I got in.

### Managing a Rust-written load balancer

4 years at Clever Cloud, a french Platform-as-a-Service provider, taught me how to make machines
talk with each other.

The company developed its own hot-reloadable load balancer, starting in 2017, and called it [Sōzu](https://github.com/sozu-proxy/sozu).
I had been developed almost single-handedly by [Geoffrey Couprie](https://github.com/Geal), the R&D person at the time.
Over my time there I [refactored and wrote some 60.000 lines of Rust](https://github.com/sozu-proxy/sozu/pulls?q=+is%3Apr+author%3AKeksoj+)
in the project, to make the code a bit more human-readable and optimize its functioning.

Over the years and under the guidance of [Florentin Dubois](https://github.com/FlorentinDUBOIS),
my responsability shifted from being a mere developer to **taking care of a load balancer infrastructure**,
making sure that routing instructions and TLS certificates were delivered to the proxies,
and that metrics were properly collected and trickled up to the kibana dashboards.

My biggest pride was my coordinating of joint efforts to **make 400 and other error pages more talkative**.
Together with the head of customer support, and my incredibly talented teammate [Éloi Démolis](https://github.com/Wonshtrum/),
we made it so that *HTTP parsing errors appear in the official Clever Cloud 400 page* (among other things).
No more wasting hours trying to replicate a user's failing request.
Just copy-paste the context provided by the error page, and we will see that the user's frontend
requested its backend with an HTTP header called `Region: Île de France`, which contains illegal UTF-8 characters.

I learned a ton about TLS certificates, protobuf, Pulsar, asynchronous coding, sysadmin,
virtualization and whatnot.

As time went by, I got overwhelmed with the infrastructure and the mission, though.
It was a lot to learn.
I guess I prefer developing, as opposed to manage-dozens-of-load-balancers.

### Redefining myself as a developer

I took some time off, walked to Santiago de Compostela, and now I am confident it's the developing I like.
But what is developing?

In short, and to quote [my former boss Quentin Adam](https://www.linkedin.com/in/waxzce/), 

> *A developer is someone who understands the customer's business, and automates it*.

Emphasis on **understand**.

I shifted my perspective, from **"code is a toy"**, to => **"code is a tool"**.

This what I now believe:

- **Language does not matter, the business matters**.
  Writing top-notch Rust or Clojure is counter-productive if the team works best with PHP.
- **The customer does not know what they want**.
  It is our duty, as developers,
  to NOT give them what they ask for, and to walk through with them what their actual needs are.
- **Once the business is well-defined, the code writes itself**. Taking the time to define
  good domain logic and data models *on the paperboard* makes writing the code easy,
  *whatever the language*.
- **The highest quality of a developer is their ability to speak to humans**.
  One must be able to explain to humans what the code does,
  and conversely, the code must be readable by humans to have any value.
  Code that is only readable by machines is technical debt.
- **AI is limited by one's technical knowledge**.
  Prompting the machine requires fulfilling all the previously defined qualities.
  AI-generated code must be proof-read by a capable human.
  Generative AI, as of 2025 and without a capable human, writes technical debt.

### Hard Skills

- **languages**: Rust is my native language, and I work with whatever is needed: Go, Java, PHP, JavaScript, Bash. It does not matter. 
- **Sysadmin**: Arch Linux, Debian, systemctl, CRON jobs, SSH, tmux
- **devops**: Docker, Docker Compose, Ansible, Github Actions, Gitlab CI
- **Protocols**: Pulsar, MQTT, RabbitMQ, TCP, HTTP, protobuf
- **Data**: PostgreQL, MySQL/MariaDB, sqlite
- **System programming**: linux sockets, linux processes, file system I/O
- **Program design**: Conceptual Data Model, Logical Data Model
- **Design patterns**: Domain-Driven Development + Dependency Inversion, mostly

### Soft Skills

- **Good with words**, written and oral, in several languages (EN FR DE EO)
- **Extraverted**. I like to reach out to other humans.
  I make sure everyone had the opportunity to speak, and that everyone understood the same thing.
- **Drawing enthusiast**. I like to draw things as I learn or explain them.
- **Markdown evangelist**. Document everything. What seems obvious today will be worth gold in two months.

### What I'd like to do

I'd like to work on any business application, on the backend side.

Get in touch [by mail](mailto:keksoj@proton.me), or see my links listed on the top of the page.



