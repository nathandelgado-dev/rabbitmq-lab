# RabbitMQ Lab with Node.js 

## Prepare project

### Install dependencies

```bash
yarn install
```

### Run RabbitMQ container (docker is required)

Rabbit on 5672 port and management on 8081.

Management user and password are "**guest**" without quotes.

```bash
docker run -d -v ${PWD}/rabbit-db:/var/lib/rabbitmq --hostname yt-rabbit -p 5672:5672 -p 8081:15672 --name yt-rabbit rabbitmq:3-management
```

## Types Exchanges

Some examples of each types exchanges:

## FANOUT

Fanout exchanges ignore Routing Keys and Patterns, send messages to all queues bound to the exchange.

## Same Exchange and Match Pattern, different queue

All subscribers receive all messages.

### Subscribers (Run commands in different terminals)

```bash
PATTERN=other_company.v2.pdf.generate QUEUE=first EXCHANGE=my-fanout node subscriber/fanout-exchange.js

PATTERN=company.v1.pdf.generate QUEUE=second EXCHANGE=my-fanout node subscriber/fanout-exchange.js
```

### Publishers

```bash
ROUTING_KEY=company.v1.pdf.generate EXCHANGE=my-fanout node publisher/fanout-exchange.js
```

## Same Exchange and Match Pattern and queue

All subscribers receive messages using round-robin.

### Subscribers (Run commands in different terminals)

```bash
PATTERN=other_company.v2.pdf.generate QUEUE=first EXCHANGE=my-fanout node subscriber/fanout-exchange.js

PATTERN=company.v1.pdf.generate QUEUE=first EXCHANGE=my-fanout node subscriber/fanout-exchange.js
```

### Publishers

```bash
ROUTING_KEY=company.v1.pdf.generate EXCHANGE=my-fanout node publisher/fanout-exchange.js
```

## Different Exchange, same Match Pattern and queue

All subscribers receive messages using round-robin.

**This happens because queue is bound to both exchanges.

### Subscribers (Run commands in different terminals)

```bash
PATTERN=other_company.v2.pdf.generate QUEUE=second EXCHANGE=my-fanout-2 node subscriber/fanout-exchange.js

PATTERN=company.v1.pdf.generate QUEUE=second EXCHANGE=my-fanout node subscriber/fanout-exchange.js
```

### Publishers

```bash
ROUTING_KEY=company.v1.pdf.generate EXCHANGE=my-fanout node publisher/fanout-exchange.js
```

## DIRECT
## Same Exchange and Routing Key, different queue

All subscribers receive all messages.

### Subscribers (Run commands in different terminals)

```bash
PATTERN=A QUEUE=first EXCHANGE=my-direct node subscriber/direct-exchange.js

PATTERN=A QUEUE=second EXCHANGE=my-direct node subscriber/direct-exchange.js
```

### Publishers

```bash
ROUTING_KEY=A EXCHANGE=my-direct node publisher/direct-exchange.js
```

## Same Exchange and Routing Key and queue

All subscribers receive messages using round-robin.

### Subscribers (Run commands in different terminals)

```bash
PATTERN=A QUEUE=first EXCHANGE=my-direct node subscriber/direct-exchange.js

PATTERN=A QUEUE=first EXCHANGE=my-direct node subscriber/direct-exchange.js
```

### Publishers

```bash
ROUTING_KEY=A EXCHANGE=my-direct node publisher/direct-exchange.js
```

## Different Exchange, same Routing Key and queue

All subscribers receive messages using round-robin.

**This happens because queue is bound to both exchanges.

### Subscribers (Run commands in different terminals)

```bash
PATTERN=A QUEUE=second EXCHANGE=my-direct-2 node subscriber/direct-exchange.js

PATTERN=A QUEUE=second EXCHANGE=my-direct node subscriber/direct-exchange.js
```

### Publishers

```bash
ROUTING_KEY=A EXCHANGE=my-direct node publisher/direct-exchange.js
```
## TOPIC

## Patterns
### Wildcards
* `*` (star) can substitute for exactly one word.
* `#` (hash) can substitute for zero or more words.

### What is a word?
The word is the string between dots: `company.v1.images.crop`

* `word.word.word.word` Each "word" string is a word.

#### You can use both
* `*.v1.#`

## Same Exchange and Match Pattern, different queue

All subscribers receive all messages.

### Subscribers (Run commands in different terminals)

```bash
PATTERN=company.v1.pdf.* QUEUE=first EXCHANGE=my-topic node subscriber/topic-exchange.js

PATTERN=company.v1.# QUEUE=second EXCHANGE=my-topic node subscriber/topic-exchange.js
```

### Publishers

```bash
ROUTING_KEY=company.v1.pdf.generate EXCHANGE=my-topic node publisher/topic-exchange.js
```

## Same Exchange and Match Pattern and queue

All subscribers receive messages using round-robin.

### Subscribers (Run commands in different terminals)

```bash
PATTERN=company.v1.pdf.* QUEUE=first EXCHANGE=my-topic node subscriber/topic-exchange.js

PATTERN=company.v1.# QUEUE=first EXCHANGE=my-topic node subscriber/topic-exchange.js
```

### Publishers

```bash
ROUTING_KEY=company.v1.pdf.generate EXCHANGE=my-topic node publisher/topic-exchange.js
```

## Different Exchange, same Match Pattern and queue

All subscribers receive messages using round-robin.

**This happens because queue is bound to both exchanges.

### Subscribers (Run commands in different terminals)

```bash
PATTERN=company.v1.pdf.* QUEUE=second EXCHANGE=my-topic-2 node subscriber/topic-exchange.js

PATTERN=company.v1.# QUEUE=second EXCHANGE=my-topic node subscriber/topic-exchange.js
```

### Publishers

```bash
ROUTING_KEY=company.v1.pdf.generate EXCHANGE=my-topic node publisher/topic-exchange.js
```

## HEADERS

## Same Exchange and Match Headers, different queue
All subscribers receive all messages.

### Subscribers (Run commands in different terminals)

```bash
HEADERS='domain=http://localhost.com' QUEUE=first EXCHANGE=my-headers node subscriber/headers-exchange.js

HEADERS='domain=http://localhost.com' QUEUE=second EXCHANGE=my-headers node subscriber/headers-exchange.js
```

### Publishers

```bash
HEADERS='domain=http://localhost.com' EXCHANGE=my-headers node publisher/headers-exchange.js
```

## Same Exchange and Match Headers and queue
All subscribers receive messages using round-robin.


### Subscribers (Run commands in different terminals)

```bash
HEADERS='domain=http://localhost.com' QUEUE=first EXCHANGE=my-headers node subscriber/headers-exchange.js

HEADERS='domain=http://localhost.com' QUEUE=first EXCHANGE=my-headers node subscriber/headers-exchange.js
```

### Publishers

```bash
HEADERS='domain=http://localhost.com' EXCHANGE=my-headers node publisher/headers-exchange.js
```

## Different Exchange, same Match Headers and queue
All subscribers receive messages using round-robin.

**This happens because queue is bound to both exchanges.

### Subscribers (Run commands in different terminals)

```bash
HEADERS='domain=http://localhost.com' QUEUE=first EXCHANGE=my-headers node subscriber/headers-exchange.js

HEADERS='domain=http://localhost.com' QUEUE=first EXCHANGE=my-headers-2 node subscriber/headers-exchange.js
```

### Publishers

```bash
HEADERS='domain=http://localhost.com' EXCHANGE=my-headers node publisher/headers-exchange.js
```

## x-match
It is required when we want to match several headers.

### Values
* **any:** get a match with only one matching header.
* **all:** get a match when all the headers match.

## x-match=any
Subscribers receive messages when any of the headers match.

### Subscribers (Run commands in different terminals)
Match `domain` but no `IP`.

```bash
HEADERS='domain=http://localhost.com&IP=76.45.140.44&x-match=any' QUEUE=first EXCHANGE=my-headers node subscriber/headers-exchange.js

HEADERS='domain=http://localhost.com&IP=76.45.140.44&x-match=any' QUEUE=second EXCHANGE=my-headers node subscriber/headers-exchange.js
```

### Publishers

```bash
HEADERS='domain=http://localhost.com&IP=76.45.140.43' EXCHANGE=my-headers node publisher/headers-exchange.js
```

## x-match=all
Subscribers receive messages when all the headers match.

### Subscribers (Run commands in different terminals)
Match `domain` and `IP`.

```bash
HEADERS='domain=http://localhost.com&IP=76.45.140.44&x-match=all' QUEUE=first EXCHANGE=my-headers node subscriber/headers-exchange.js

HEADERS='domain=http://localhost.com&IP=76.45.140.44&x-match=all' QUEUE=second EXCHANGE=my-headers node subscriber/headers-exchange.js
```

### Publishers

```bash
HEADERS='domain=http://localhost.com&IP=76.45.140.44' EXCHANGE=my-headers node publisher/headers-exchange.js
```
