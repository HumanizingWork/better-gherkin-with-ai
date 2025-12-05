# Gherkin Code Review Instructions
This file defines instructions for code reviews of Gherkin (`*.feature`) files, per the recommendations in "Behavior-Driven Development with Cucumber" by Richard Lawrence and Paul Rayner (bddwithcucumber.com).

## Most Gherkin Examples on the Web Are Bad
Forget what you've inferred from Gherkin examples on the internet. Most examples in the wild fail the advice below. Focus on the advice and examples here for your reviews.

## Scenarios Should Be Concrete Domain Examples
Good Gherkin scenarios are concrete domain-language examples of the behavior of a system. This is a good example (from a suite describing a public library website):
```gherkin
Scenario: Search for a book 
  Given the following catalog: 
      | Title                | Author             |
      | A Tale of Two Cities | Charles Dickens    |
      | Crime and Punishment | Fyodor Dostoyevsky |
      | East of Eden         | John Steinbeck     |
  When I search for "Dickens"
  Then I should see only "A Tale of Two Cities" in the results
```

Here's an example scenario for the same behavior that would be considered "Tautological." Suggest that tautological scenarios get more details so they become a real domain example. 
```gherkin
Scenario: Search for a Book
  When I search for a book
  Then I should see the correct results 
```

The following example is "Overly Technical." It has too many implementation details such as the database keys, XPath, and CSS, as well as phrases like "fill in" and "click". Suggest that those implementation details move into step definitions and get summarized into domain concepts. 
```gherkin
Scenario: Search for a book
  Given the following Authors:
    | id | name               |
    | 1  | Charles Dickens    |
    | 2  | John Steinbeck     |
    | 3  | Fyodor Dostoyevsky |
  And the following Titles:
    | id | title                | author_id |
    | 1  | A Tale of Two Cities | 1         |
    | 2  | Crime and Punishment | 3         |
    | 3  | East of Eden         | 2         |
  And I'm on "http://www.bookstore.biz/"
  When I fill in "Dickens" in "//input[name()='q']"
  And I click "#searchButton"
  Then there should be 1 "div.book div.title" element
  And the page should contain "A Tale of Two Cities"
```

The following example is "Scripty." It reads like a manual test script. Phrases like "fill in" and "click" are clues for Scripty scenarios, as are multiple `When` steps. Suggest summarizing groups of steps into domain concepts.
```gherkin
Scenario: Search for a book
  Given the following catalog:
    | Title                | Author             |
    | A Tale of Two Cities | Charles Dickens    |
    | Crime and Punishment | Fyodor Dostoyevsky |
    | East of Eden         | John Steinbeck     |
  And I'm on the bookstore home page
  When I fill in "Dickens" in the search field
  And I click "Search"
  Then I should see only "A Tale of Two Cities" in the results
```

The following example is using domain language but it has "Excessive Detail" (the publisher, ISBN, format, and price). Suggest removing details that don't help express the example behavior the scenario describes. 
```gherkin
Scenario: Search for a book
  Given the following catalog:
    | Title                | Author             | Publisher         | ISBN       |
    | A Tale of Two Cities | Charles Dickens    | Qualitas Classics | 1897093594 |
    | Crime and Punishment | Fyodor Dostoyevsky | Simon & Brown     | 1936041030 |
    | East of Eden         | John Steinbeck     | Penguin           | 0142000655 |
  When I search for "Dickens"
  Then I should see only the following in the results:
    | Title                | Author          | Format    | Price  |
    | A Tale of Two Cities | Charles Dickens | Paperback | $14.90 | 
```

## Other Things to Consider at the Feature or Scenario Level
Lots of data setup in a `Background` section, especially with many steps and multiple tables, may indicate:
1. The feature specifies too many things and could be split, especially if some of the data is used by some scenarios and other data by other scenarios.
2. It could be useful to push some of the data setup into hooks or support code and give entities names (i.e., create "data personas" or "named fixtures"). 
3. Some of the data might be there because a form or API requires it but not because it helps express the behaviors in the feature. In that case, push the default data into the automation code and only include it in scenarios to override the defaults.

Groups of steps (taking variable data into account) that appear together frequently might indicate the need for a larger domain concept. For example,
```gherkin
Given the routing number is 123456789
And the account number is 700800900111
When the agent validates the bank account
```
could be expressed more concisely as:
```gherkin
Given the routing number is 123456789
And the account number is 700800900111
When the agent validates the bank account:
  | Field    |Value|
  | Routing Number | 123456789    |
  | Account Number | 700800900111 |
```

Scenario titles should express what's unique about the scenario. If using `Rule`s, Scenario titles don't need to restate the rule but should express the variations under the rule. If not using rules, Scenario titles should concisely express both the rule and the variation covered in the scenario. 

Be consistent about the actor throughout scenarios—use either first person ("I") or third person ("a patron", "the user") within a feature file, but don't switch between them arbitrarily. For first person, it usually makes sense to have a `Given` step, often in a `Background`, that declares who "I" refers to (e.g., `Given I'm logged in as a librarian`).

## Advice for Language in Steps
Once the scenario is at the right level of abstraction—a concrete example in domain language. Highlight the following in step language:

* `Given that` can often be shortened to just `Given`. **Exception:** "that" is not a filler word when it's a demonstrative pronoun referring to something previously mentioned (e.g., "that book" referring back to a book from a prior step)
* Likewise, remove any filler words that don't do work
* Use present perfect tense in `Given` steps when describing actions that have been completed (e.g., "a patron has checked out a book"). Active voice when the actor matters, passive voice when it only matters that the system/entity is in a certain state. Use present tense for describing current state (e.g., "the book's status is 'Available'"). It's just context—who cares how it got that way?
* Use present tense, usually active voice, in `When` steps. Something’s happening, and someone is making it happen.
* The actor in a when step should usually be a human user. Occasionally, however, the action is a change in the state of a system or even a change in the date or time. 
* Use conditional, passive voice in `Then` steps. You’re describing objectively how the state of the system should be now.
* Prune language about feelings and desires (other than "should" and "should not").
* Use a colon before a table or multiline-string arguments to suggest that the reader should continue reading the table as part of the step
* Focus domain terms—use the same word or phrase for the same thing every time, and use that word or phrase to only refer to one thing. (Cucumber helps enforce this via global step definitions.)
* Be consistent about case, punctuation, etc. Variation without meaning wastes the reader’s brainpower.
* Avoid haphazard capitalization of common nouns.
* **Exception:** Capitalize proper domain terms that represent specific types or categories in the system (e.g., "Book on CD" as a library item type)

## Response Format
1. State the problem (1 sentence)
2. Why it matters (1 sentence, if needed)
3. Suggested fix (snippet or specific action)

## When to Stay Silent
If you’re uncertain whether something is an issue, don’t comment. 