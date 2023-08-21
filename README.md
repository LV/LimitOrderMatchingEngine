# Matching Engine Specification

_The following specification was taken from the now defunct [quantcup](https://web.archive.org/web/20120110020402/http://www.quantcup.org/home/spec)._

<br>

## Summary

A price-time priority limit order matching engine is responsible for determining when buy and sell orders transact (see Definitions below if unclear).

High frequency trading has pushed the limits of legacy matching engine implementations at the NYSE, CME, and other major exchanges. Proprietary trading groups can benefit from an internal matching engine to trim execution costs. Electronic market makers need to locally reconstruct the order book to determine when to provide liquidity. At the core of all these is the matching engine you will be writing.

Implement the interface provided in `engine.h` in a file `engine.c` for your entry. An example `engine.c` is provided. Rename or delete it before writing your own.

_Language: C (gcc 4.4.5)_

Starter Code: [match.zip](https://web.archive.org/web/20110722152203/http://dl.dropbox.com/u/3001534/match.zip)
    Includes: API interface, autotester, makefile, simple example solution, type definitions, assumptions.

<br><br>

## Starter Code Descriptions

### ___engine.h___

<dl>
  <dt><code>init()</code></dt>
  <dd>Called once before the first orders are submitted. Initialize your data structures here. Set the next order id to 1.</dd>

  <dt><code>destroy()</code></dt>
  <dd>Called once after the last order is submitted. Free dynamically allocated memory here and cleanup. Calling init() after destroy() should completely reset the matching engine.</dd>

  <dt><code>limit()</code></dt>
  <dd>Called when a trader submits a new order. Check whether the order crosses with any existing orders. If so, call execution. If not, queue the order up with the others that have been submitted. Return the next order id and then increment it. The first order id returned is 1.</dd>

  <dt><code>cancel()</code></dt>
  <dd>Remove an order from the queue, identified by the order id returned from limit().</dd>

  <dt><code>execution()</code></dt>
  <dd>Report that a trade occured. Called twice whenever two orders cross, once with the buyer's name and side, and also with the seller's name and side. The two calls to execution may be in any order.</dd>
</dl>

<br>

### ___types.h___

<dl>
  <dt><code>t_orderid</code></dt>
  <dd>Order identification numbers which are returned by limit(). Used to uniquely identify the order for the sake of cancellation.</dd>

  <dt><code>t_price</code></dt>
  <dd>Price of a share. In the range 0-65536, and interpreted as divided by 100. E.g. the range is 000.00-655.36. E.g. the price 123.45 = 12345; the price 23.45 = 2345; the price 23.4 = 2340</dd>

  <dt><code>t_size</code></dt>
  <dd>Number of shares.</dd>

  <dt><code>t_side</code></dt>
  <dd>Boolean representing whether a bid (0) or ask (1).</dd>

  <dt><code>t_order</code></dt>
  <dd>Order submitted to the matching engine containing symbol, trader name, side, price, and size.</dd>

  <dt><code>t_execution</code></dt>
  <dd>Execution report sent to trader informing them of the symbol, side, price, and size of the transaction.</dd>
</dl> 

<br>

### ___limits.h___

<dl>
  <dt><code>MAX_PRICE</code></dt>
  <dd>Assume <code>limit()</code> will never be called with an order whose price field is greater than this number.</dd>

  <dt><code>MIN_PRICE</code></dt>
  <dd>Assume <code>limit()</code>code> will never be called with an order whose price field is less than this number.</dd>

  <dt><code>MAX_LIVE_ORDERS</code></dt>
  <dd>Assume there will be no more than this number of uncrossed orders sitting on the book at any time.</dd>

  <dt><code>STRINGLEN</code></dt>
  <dd>Assume the trader names and order and execution symbols will be this long.</dd>
</dl>

<br>

### ___Makefile___

<dl>
  <dt><code>all</code></dt>
  <dd>build the implementation in <code>engine.c</code> and the test script. Then call <code>./test</code> to run the auto testing script.</dd>

  <dt><code>clean</code></dt>
  <dd>remove <code>engine.o</code>, executables, and emacs backups</dd>
</dl>

<br>

### ___derived.h+c___
Not required. These are provided to show you how one can derive interesting order types that are offered by many exchanges from the basic `limit()` and `cancel()` API.

<br>

### ___engine.c + double_link_list.c___
An example implementation that is not optimized. It treats the bid and ask queues as doubly linked lists. This makes `limit()` O(n), cancel() O(n) and in addition the constant factors, ignored by big-O, are quite high.

<br>

### ___test.c___
Script to automatically test your implementation's logic.
After implementing `engine.c`, call:
```bash
$ make all
$ ./test
```
Fix the errors before submission or your entry will be disqualified. Tests are incremental so they can be used to check implementation progress. 

<br>

### ___score.c___
Script to automatically test your implementation's speed. Shows how the scoring script works and what you should optimize for. This will be run on an isolated core (`isolcpus=1`).
After implementing `engine.c`, call:
```bash
$ make all
$ taskset 2 ./score
```
Try to minimize both the mean and the standard deviation of the latency.

<br>

### ___score_feed.h___
Simulated order and cancel data feed for use with `score.c`. This is an example of what the official scoring dataset will look like. Orders with `price = 0` correspond to cancels with `orderid = size`.

<br><br>

## Scoring
<s>Submit only your engine.c file by emailing it to <b>[REDACTED]</b> with the subject line: QuantCup1: *[Displayed Name]*. I will verify *[Displayed Name]* with the email address you registered Your registration information must be accurate to be scored. You may submit additional code but not submit modified versions of any of the provided code except for engine.c. For example one may submit engine.c and double_link_list.c since it does not require modification of Makefile.</s>
Only the `engine.c` file is to be modified

Minimum Requirements to be Considered:
    1) Implement all the matching engine logic
    2) Compile without errors on Ubuntu 10.10 and run using dedicated core: `isolcpus=1` and `cpu affinity`
    3) Pass all the logic tests in the autotester script
    4) Include comment explaining overall architecture and main optimizations

Score Formula = mean(latency) + sd(latency), as reported by `./score`. 
    `./score` will use a different `score_feed.h` than the one provided to discourage over-optimization.
    The entry with the lowest score wins.

Disqualification:
    You may be disqualified, without the judge providing justification nor entertaining appeals, for any perceived attempt to cheat or otherwise game the contest.

[Detailed information about judging platform](https://web.archive.org/web/20110722152203/http://www.quantcup.org/home/judgeplatform)

Suggestions
Data Structure:
    [How to Build a Fast Limit Order Book](https://web.archive.org/web/20110722152203/http://howtohft.blogspot.com/2011/02/how-to-build-fast-limit-order-book.html) ([copy](https://web.archive.org/web/20110722152203/http://www.quantcup.org/home/howtohft_howtobuildafastlimitorderbook)) from WK Selph's HowToHFT Blog
Cache Optimization:
    [Building a HPC Financial Exchange Video](https://web.archive.org/web/20110722152203/http://www.infoq.com/presentations/LMAX)

<br>

## Definitions
**Price-Time**: At any time there are multiple buyers waiting for someone to come sell shares at an acceptable price. Price-time priority means that the buyers will get filled in order of who offers the best price, and if the prices tie, then by who has been waiting the longest. The same prioritization applies to the sellers who submitted orders that did not fill immediately. Price-time is how the NYSE, NASDAQ, CME, and other major exchanges work.

**Matching Engine**: the computer program at the heart of an exchange which (1) stores all the outstanding limit orders (this data structure is called an "order book" or just "book"), (2) accepts new orders, (3) determines if new orders cross with any orders already on the book, (4) notifies both sides that their orders have been executed if it does cross, (5) queues orders in the book if they do not cross, (6) accepts order cancellation requests.

**Bid**: buy limit order

**Ask**: sell limit order

**Cross**: a bid and an offer having mutually acceptable prices. A cross results in a trade occurring and shares being transferred. For example a bid for 100 shares at $100 on the book and a newly arriving ask for 50 shares at $100 dollars cross at the price of $100, resulting in the trade of 50 shares for $100. The same trade would occur if the newly arriving ask has a price of $99, or even less.

**Execution**: short for the execution report which is sent to a trader notifying them that a trade has occurred, how many shares have been transacted, and at what price. Executions are always sent in pairs, one to the buyer and one to the seller.

**Latency**: the time a computer process takes from start to finish. Can be broken down into matching engine, connection, network stack, data processing, and decision logic latency. In this challenge you will be optimizing matching engine latency only.

**Jitter**: roughly the standard deviation of latency. High jitter is bad because trades will nondeterministically be delayed.

<br><br>

## FAQ
**Q**: I assume that writing code in multiple files or some other language, compiling them, and sticking the source and a giant assembly dump in `engine.c` wouldn't be appreciated?

**A**: Correct. Code must be portably written in pure C.

<br>

**Q**: `limits.h` has the following: `#define MAX_LIVE_ORDERS 65536`. `MAX_TRADERS` and `MAX_SYMBOLS` must therefore be `<= 65536`; what is the upper limit for these?

**A**: `MAX_TRADERS = 50`. `MAX_SYMBOLS = 1`.

<br>

<s><b>Q</b>: A good implementation of an order matching engine would be worth far more than 10k. Asking for the rights to the code seems a bit much. Can you remove the phrase, "Participant grants Max Dama a perpetual, irrevocable, worldwide, royalty-free, and non-exclusive license to use, reproduce, publicly perform, publicly display and create a derivative work from, any Entry that Participant submits to this Challenge."

<b>A</b>: That would expose me personally to the risk of an IP suit if I later work on anything related in the future. I doubt it is worth "far more than 10k", otherwise I would not open source the Entries. Every HFT firm already has their own implementation (or multiple for different exchanges) and I'm sure theirs are as good as anything that might be submitted since they hire people like the contestants and pay them to do this full time. It's a common illusion among the trading community that one's own work is pricelessly valuable. You could spend some time working on an Entry, and if you get to a point where you have optimized so well that you think it is worth 10k, you could try selling it and waiting to submit until the contest deadline in August if it turns out not to be as valuable as you thought.</s>

<br>

**Q**: Is there a max number of orders per test, such as `MAX_ULONG`? This would negate the need of a modulus operation.

**A**: `1,000,000`

<br>

**Q**: Do we have to take into account the case where `orderid` overflows and a limit is asked for an `orderid` that already exists?
This would happen, for example, when an order is never filled.

**A**: No. This will not happen

<br>

**Q**: May I submit multiple Entries?

**A**: Yes. Only the latest one will be counted. You may not receive a score for a few days in order to discourage rapid multiple submissions. 

<br>

**Q**: Is inlined assembly code allowed?

**A**: No. The idea is to have a platform independent solution, so we cannot accept inlined assembly.
