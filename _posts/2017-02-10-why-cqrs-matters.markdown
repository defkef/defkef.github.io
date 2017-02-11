---
layout: post
title:  "Why CQRS matters in software design"
date:   2017-02-10
share: y
disqus: y
---


## Unified data structures are a problem of MVC applications

In MVC applications data structures are primarily associated with the persistent domain models like Rails’ ActiveRecord. These models form the basis of most applications on which major functionality is built upon.

However, I think it is worth to differentiate data structures especially in read and write models (CQRS), as it encourages better software design. CQRS stands for Command Query Responsibility Segregation.

> At its heart is the notion that you can use a different model to update information than the model you use to read information. Read more by [Martin Fowler](https://martinfowler.com/bliki/CQRS.html).

To illustrate the benefits of that pattern I consider the data flow from the input to the output of a grown Rails (or other MVC) application, which often looks like this:

<div class="mermaid" style="height: 500px; text-align: center;">
graph TD;
    A[Input]==>B[Transformation];
    B==>C[Domain Models];
    C==>D[Transformation];
    D==>E[Output];
</div>

Typically the input (forms, file uploads, etc.) is parsed or mapped onto multiple domain models in order to fit a persistent database scheme. Before the data can be presented back to the user another transformation adapts the persistent models such as formatting values or combining several models under a common interface.

In this approach the domain models get caught between two transformation steps and must satisfy the read and write operations of your system. In a grown application with many requirements these models kind of seem arbitrarily chosen and are not well adjusted to a specific use case anymore. They become more bloated feature after feature and are hard to maintain and test.

The reason behind this, to some extent, is that it is easy to mix up read (query) and write (command) operations and their underlying data structures respectively. Thoughts like *“Here I am going to need attributes A and B and therefore I need X and Y”* facilitates tight coupling and weak cohesion in any applications. There is no such thing as a common data structure.

I will try to make it clear by an example. Suppose we need to develop a simple app to create and list invoices of our customers. I came up with a (simplified) command *CreateInvoice* of 3 parameters:

```
CreateInvoice :: date, amount, customer_name -> _
```

And a query *ListInvoices* with a time period as parameter and a list of corresponding invoices as their result.

```
ListInvoices :: time_period -> [invoice_number, date, amount, customer_name]
```

The domain model in a typical MVC application might look like this:

| Invoice        |
| ------------   |
| invoice_number |
| data           |
| amount         |
| customer_name  |

To satisfy both requirements all we have to do is to store the input into the invoice model and fetch all invoices of a given period. The concrete implementation is not only trivial because of a single model involved, but also the systems input conforms directly to its output, i. e. the read and write model are identical.

Then a new requirement pops up and we need to keep track of the payment status as well. So I add another command:

```
PayInvoice :: invoice_number, date -> _
```

And adjust the query to

```
ListInvoices :: time period ->  [invoice_number, date, ..., paid_at]
```

So what are the choices for the implementation?

## Change the existing model

In most MVC applications the domain models are tightly coupled to the persistence layer and hence are more related to the output than to the input of the system. This is mainly for efficiency reasons since reading occurs generally more often than writing.

That said, we add another attribute to the invoice model to reflect the interface of *ListInvoices* like this:

| Invoice        |
| ------------   |
| invoice_number |
| data           |
| amount         |
| customer_name  |
| paid_at        |

The parameter of the new command *PayInvoice* is stored into the invoice model. The output code keeps nearly unchanged, since it is still a 1:1 mapping from that model. We just need to add another attribute to the view layer.

Well, because that change is convenient, **it is quite problematic** in every sense. In contrast to the original example the input, fed by two different operations, does not conform 1:1 to the output anymore. Therefore it should not be unified in a common data structure either. In the above implementation, creating and paying invoices depend on the same domain model, which leads to coupling between both commands through the Invoice model. A change in a requirement is likely to provoke extensive change in the code of rather unrelated requirements due to the strong coupling.

Furthermore the act of payment is transformed to fit the read model early on. You might lose information which might be useful in future requirements. For instance the payment could have been accompanied by metadata, which then needs to be stored in additional attributes as well but has nothing to do with invoices.

Going with this approach adding more features (and input) to the system would require you to change existing domain models again and again. This is not only a source for defects, these models also get bloated really quickly because they are influenced by all kind of use cases.

## Add a second write model

Probably you already guessed it: Payments are a concept on their own. Therefore we introduce a Payment model to separate both concerns.

| Payment        |
| ------------   |
| invoice_number |
| date           |

Now the write operations *CreateInvoice* and *PayInvoice* are fully separated by their corresponding write models Invoice and Payment. A potential change in one does not affect the other.

With this approach the view code gets a little more complex, since *ListInvoices* needs to know about both inputs of your system, i. e. about invoices and payments.

However, Rails for example hides the complexity by relationships (has many, belongs to, etc.) defined between those models which leads to an entangled gigantic domain model. I find this approach problematic since it introduces query demands into the write model. Associations (only needed for querying) are already established on the write side. A change for example in the composition is likely to influence commands as well.

## Dedicated write and read models

The relation between invoices and payments should only exist in terms of the query mentioned above. To address this issue we introduce a read model which provides the necessary data. We keep both write models isolated and can get the exact same interface as in our first approach.


| InvoicesWithPaymentState |
| ------------   |
| invoice_number |
| data           |
| amount         |
| customer_name  |
| paid_at        |

Stay with me for a moment. It needs to be clear that the (persistent) write models are not influenced by queries, only by their corresponding command. On the other side, read models should only be influenced by their corresponding queries or projections. All information needed for output are deduced from the write models and reduced into a corresponding read model. The transformation takes place in between those models.

<div class="mermaid" style="height: 500px; text-align: center;">
graph TD;
    A[Input]==>B[Write model];
    B==>C[Transformation];
    C==>D[Read model];
    D==>E[Output];
</div>

Just to be clear ...

… one more requirement pops up.

Our boss needs us to import invoices and payments from another system. They will come in as a CSV file with slightly different attributes.

Again, a naive approach would be to map that data onto our existing Payment and Invoice models. Instead, I would argue we should go for a third data structure that reflects the CSV file's attributes or the file altogether. The transformation which creates the read model is going to be extended to take that third model into account. The read model itself does not change and write models are not cluttered by a potential origin attribute to indicate where the invoice or payment came from.

## Conclusion

An application's output is rarely structured as its input and therefore should not be mixed up in the code either. Splitting the functionality into queries and commands was on purpose and should be materialized by their corresponding read and write models.

At best you have dedicated data structure for every operation and no transformations involved before the input hits the database. That way we do not lose information in the first place. It is much easier to deduce information needed for output from original event happened. In order to add functionality one might need to add new models and transformations but seldom change existing models. To optimize for performance you might introduce a caching layer or cached read models.

One idea that brings that a step further is event sourcing, where you have basically a chain of events as your write model. The different read models are projected by computing these events successively.

Read more on:

* [http://oierud.net/bliki/EventSourcingInRuby.html](http://oierud.net/bliki/EventSourcingInRuby.html)
* [https://martinfowler.com/eaaDev/EventSourcing.html](https://martinfowler.com/eaaDev/EventSourcing.html)
* See also [Greg Young's talk](https://www.youtube.com/watch?v=JHGkaShoyNs) about CQRS and Event sourcing


<script src="{{ site.url }}/javascripts/mermaid.min.js"></script>
<script>
$(document).load(function() {
  mermaid.initialize();
});
</script>
