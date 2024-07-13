# Event Query Language Snippets

This document contains a collection of Event Query Language (EQL) snippets that can be used to help in hunting and detection scenarios.
The document is written with the EQL version implemented in Elastic Security, but can be adapted to other EQL implementations as well.

## Sequence

### Initial Sequence Definition

- The start of the sequence is defined by the keyword `sequence`.
- Usually it makes sense to use the by statement to group the sequence by a specific field. For example user.name or host.id. It is also possible to group by multiple fields.
- The `with maxspan` statement defines the maximum time span between the events in the sequence. It makes sense to always use that statement.

example: ``sequence by host.id, source.ip, user.name with maxspan=15s``

### Defining a Sequence

Once the Sequence header is defined, the sequence is defined by a list of at least two events that should appear in the specified order.
Events are defined by brackets and at least a where statement.
For example ``[ authentication where host.os.type == "linux" ]``
The part before where is the event.category in Elasticsearch, so either use the the propper category, or if unsure you can use "any" as a wildcard.
The part(s) after where are the conditions that should be met for the event.
After the brackets it's possible to add additional conditions:

#### by

With that statement you can introduce constraints from one event to the next. For example ``by process.id`` and then in the next event it would be ``by process.parent.id``
For example this query will look for the process cmd.exe, and then look if the process.id matches the next event which is a file creation event by the process.parent.id.

```eql
[ process where process_name == "cmd.exe" ] by process.id
[ file where event.action == "create" ] by process.parent.id
```
#### with runs

With the ``with runs`` statement, you can define how many times the same event has to repeat. For example ``with runs = 10`` will look for the same event 10 times in a row, still constrained by the maxspan of course.
This is for example interesting to look for a brute force attack, where the same event is repeated multiple times.

```eql
sequence by host.id, source.ip, user.name with maxspan=15s
[ authentication where event.outcome == "failure" ] with runs = 10
```