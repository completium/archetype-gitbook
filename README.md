# What is Archetype

Archetype is a domain-specific language \(DSL\) to develop smart contracts on the [Tezos](https://tezos.com/) blockchain, with a specific focus on contract security.

{% embed url="https://medium.com/@benoit.rognier/archetype-a-dsl-for-tezos-6f55c92d1035?source=friends\_link&sk=aff026858b2fad66f2251da87ec6e881" caption="Medium article" %}

Archetype is funded by the [Tezos Foundation](https://tezos.foundation/) and developed by [edukera](https://www.edukera.com).

## Smart contracts

[Smart contracts](https://en.wikipedia.org/wiki/Smart_contract) are programs that are executed on the [blockchain](https://en.wikipedia.org/wiki/Blockchain). They were popularised in 2015 by Ethereum. They allow us to read and write simple data on the blockchain. 

Smart contracts unleash the full potential of the blockchain because they enable the development of a new class of application, called [Dapp](https://www.youtube.com/watch?v=CDQX8inMCt0), which benefits from blockchain's strengths \(decentralized, trust-less, immutable, governed by consensus in Tezos case, ...\). 

A smart contract is similar to a [stored procedure](https://en.wikipedia.org/wiki/Stored_procedure) on a public distributed database. As such, it must ensure the **logical consistency and integrity** of the data.

## What's the problem?

A smart contract is a _standard_ program, and, as such, it does not come with any guarantee that it will function correctly. We all have in mind the [TheDAO incident](https://www.vice.com/en_us/article/qkjz4x/thedao): a bug in the smart contract made it possible to withdraw several times the money you would put in the contract. As a result, more than 50 million dollars were lost.

It’s a highly non-trivial question to decide whether a program is correct or not, even for the programmer. The best practice is to develop a set of tests the smart contract must pass before deployment. However, testing is limited by the capacity to figure out all possible situations of execution. 

## Required services

In order to circumvent the inherent potential inconsistency of the smart contract, several services are required to increase the _apriori_ level of confidence you may have in a smart contract, before deploying it.

#### Verify

Program verification is the gold standard when it comes to trust-less software quality.

What is program formal verification?

Say you have a program and a specific property that the program is supposed to have; say this property is written in formal logic. Program verification consists in figuring out a **mathematical proof** that the program has the property.

A mathematical proof is the perfect trust-less guarantee of the program quality, because the only thing you need is to check that the proof is correct, and this question is known to be _decidable;_ decidability means here that it is possible to have a computer check the correctness of the proof _automatically_.

Are there limits? 

Yes of course. 

First, the guarantee you get is limited to the properties you have been able to identify. A critical property may still be forgotten!

Second, you need to be skilled in _formal methods_ to formalize the contract properties, and even more to prove them, especially if they are complex.

#### Test

When program verification is too complex or too expensive, it is necessary to set up test batteries to provide standard quality insurance.

#### Simulate

Observing a contract in action is a quick way to gain a reasonable level of insight into the behavior of a smart contract.

#### Document

It is always good to read what the smart contract designer thinks about what it is supposed to do! If it is aligned with the insights provided by program verification and simulation, it will reinforce the confidence you have in the contract.

#### Execute

This last service is obvious and has to do with the possibility to execute the smart contract on the blockchain... Without this service, the others would not be relevant.

## Solution frameworks

For each of the services identified above, an existing framework has been selected. A compromise between _ease of use_ and _performance_ was the selection criteria. 

This selection is not definitive nor exhaustive. It should be considered as a starting point.

The following table shows the selected solution for each service and the main benefits from it:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Service</th>
      <th style="text-align:left">Solution</th>
      <th style="text-align:left">Benefits</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Verify</td>
      <td style="text-align:left"><a href="http://why3.lri.fr/">Why3</a>
      </td>
      <td style="text-align:left">
        <p>High level of automation.</p>
        <p>Why3 is a verification framework based on Hoare Logic. It generates verification
          tasks for external SMT solvers (alt-ergo, CVC, Z3, ...). Verification tasks
          may also be exported to <a href="https://coq.inria.fr">coq</a>.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Test</td>
      <td style="text-align:left"><a href="https://www.cs.tufts.edu/~nr/cs257/archive/john-hughes/quick.pdf">Quick-check</a>
      </td>
      <td style="text-align:left">Automated random test generation</td>
    </tr>
    <tr>
      <td style="text-align:left">Simulate</td>
      <td style="text-align:left"><a href="https://gsuite.google.com/intl/en_za/products/sheets/">Google spreadsheets</a>
      </td>
      <td style="text-align:left">Cloud-based spreadsheet with complete scripting capability</td>
    </tr>
    <tr>
      <td style="text-align:left">Document</td>
      <td style="text-align:left"><a href="https://en.wikipedia.org/wiki/Markdown">Markdown</a>
      </td>
      <td style="text-align:left">Easy to write and read; easy to convert to other formats (HTML, pdf, ...)</td>
    </tr>
  </tbody>
</table>

{% hint style="warning" %}
'Test' and 'Simulate' services will be available in 2021.
{% endhint %}

The purpose of the archetype Test service is not to replace the standard test approach, but rather to enhance it with random test generation. The idea is to leverage formal properties to derive random and relevant tests.

## A single language

Each framework identified in the previous section comes with a specific language:

| Solution | Language | Type of language |
| :--- | :--- | :--- |
| Why3 | Whyml \(\*.mlw\) | ml language, like Ocaml,  with specific instructions for verification |
| Google spreadsheets | Google script \(\*.gs\) | Similar to javascript |
| Document | Markdown \(\*.md\) | plain text with minimalist formatting  instructions for maximum readability  \(unlike HTML markup tags\) |

As a consequence, in order to benefit from these frameworks, you need to develop several versions of the smart contract, one for each framework.

The fundamental issue, beyond time and skills, is the issue of logical consistency between each version. 

How to make sure that the version for formal verification is consistent with the version for execution? Some verification frameworks have their own solution \(namely extraction\), but what about the consistency over the entire set of services?

Hence the need for a ****single language to describe the business logic of an archetype contract, from which the different operational versions may be derived.

![](.gitbook/assets/targets.png)

{% hint style="info" %}
In a nutshell, Archetype is a programming language that is transcoded to target languages for specific services.
{% endhint %}

As such, archetype may also serve as a target language for existing language to benefit from archetype services.  

Follow the link below to read more about the archetype language features.

{% page-ref page="archetype-language/" %}

## Contract library

Archetype comes with a library of several dozen contracts. Its purpose is to identify typical contracts to illustrate the archetype language, and bootstrap or inspire your development.

The library covers some key blockchain processes :

* purchase transaction \(escrow\)
* decision process \(voting, auction\)
* investment \(tokens, market place, financial notes\)
* insurance
* IoT
* other frameworks example for comparison \(Hyper ledger, clause.io, ...\)

Some of these contracts come with relevant properties. Feel free to submit any new property you may find relevant.

