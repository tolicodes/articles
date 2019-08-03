# Modern TDD - Storybook, Jest, Pickle - Cypress + SauceLabs

## Why testing is important
You join a 5 person startup. The business plan isn't quite fleshed out. You're probably changing direction every few months. There's no time for testing. The only mission is to survive. After all, testing it time consuming, time is money, and right now you every penny counts.

Before you know it, your 5 person startup baloons. Now you have an active sales team, targeting large corporations, and an engineering team to match. Despite your engineering team quadrupling, you seem to be writing new features at nearly the same rate as before. Your team is spending more time fixing bugs than it is expanding your business.

I've been through this process more times than I could count. Especially in sales driven organizations, pumping out code at top speed is  the prioirity, all at the cost of creating tremendous technical debt. 

There are a few companies that I've worked at, such as [HOVER](http://hover.to) and [Smartly](http://smart.ly), which prioritized testing early on in the process and reduced their technical debt significantly. 

It's difficult to convince management to take on a TDD approach - after all there's no direct revenue tied to tests. But there's quite a business case for doing so:

1. **Reduce future bugs**: Untested code creates a significant amount of bugs, which take a lot of time and money to eliminate
2. **Technical debt**: Untested code creates a lot of technical debt, that is, inefficient and poorly documented code that makes writing new features difficult
3. **Improve communication and create concise code**: TDD is a change in how you think. It forces you to slow down and communicate clear requirements with the product team for how the feature should work.
4. **Reduces fear of refactoring**: It is significantly safer to refactor well tested code. By having Unit, Feature, and End to End test suites you virtually guarantee that critical functions will not break
5. **Automatic Documentation**: A well written test suite documents the tests from a functional (Unit Test), feature (Feature Test), and user flow (E2E) perspective. This makes it easy to onboard new engineers, designers, and other product employeees.
6. **Visualize your coverage/risk**: Unit Testing tools include a visualization feature to see what parts of your code are undocument, and therefore hold the most risk

## Types of Testing
### Unit Test
A Unit Test tests individuals compontents, classes, and functions. An example of a unit test would be something that tests a business logic class:

```
class Calculator {
  add (a, b) {
    return a + b;
  }
}

test(Calculator, () => {
	test('add', () => {
		it('adds two numbers', () => {
			
			expect(
        });
    });
});
```

### Feature Test
### End to End / Integration Test
### Browser and Device Testing

## Testing basics
### Have at least a basic test

## Making a plan
### Writing your test cases / stories first
### Breaking things down into story points

## Storybook based development
### Why develop in isolation
### Setting up storybook
### Building a StyleGuide
### Converting your test cases into Storybook stories
### Decorators (wrapping your stories)
#### Snapshot test your stories (Storyshots)
### Addons
####  Knobs
####  Actions Logger

## Jest Tests
### Why do unit tests
### Jest setup
### Basic test
### Setup and Tear down (beforeEach, afterEach)
### Snapshot Testing
#### Good 
#### Bad
### Enzyme
### Mocking components
#### what to mock and what not to mock
#### Testing connected components
#### Testing functions and callbacks
#### spies (jest.fn())
##### toHaveBeenCalled
##### toHaveBeenCalledWith
### Testing redux sagas

## Coverage
##### CodeCov 

## CircleCI setup

## End-2-End Testing

## Cypress Tests
### Selenium / SauceLabs Tests
### Pickle - what it does and how to use it
<!--stackedit_data:
eyJoaXN0b3J5IjpbOTAwNDYyNTY1LDEzOTU0MDE0OTcsODA1NT
gzNTA3XX0=
-->