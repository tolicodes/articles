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
A Unit Test tests individuals compontents, classes, and functions. These tests are done in complete isolation from the app environment.

Unit Testing Frameworks include:
- Jest
- Jasmine
- Mocha

An example of a Unit Test (written in Jest) would be something that tests a business logic class:

```
class Calculator {
  add (a, b) {
    return a + b;
  }
}

test(Calculator, () => {
	test('add', () => {
		it('adds two numbers', () => {
			const calc = new Calculator();
			expect(calc.add(1, 2)).toBe(3);
        });
    });
});
```

### Feature Test
A Feature Test tests a specific feature in the app environment. For a Feature Test to occur, you must fully load the frontend of the app, but you can mock the backend response (or response from another system).

Feature Testing frameworks include
- Selenium
- WebDriver.io
- Cypress
- TestCafe

There are also abstractions on each of these frameworks, which allow you to write the tests in plain English. I wrote one such framework, [PickleJS](http://picklejs.com)

Note that you will need to write mocks for each API request your app makes, because a Feature Test should be completely independent from the Backend systems. Essentially, it should not test any part 

An example (using Cypress):

```
describe('Cart Checkout', () => {
	it('should be able to check out', () => {
		cy.click('.checkout-button')
		cy.location('pathname').should('eq', '/checkout')
	});
});
```

And abstracted in PickleJS:

**cart.feature**
```
Feature: Cart Checkout
	Scenario: I should be able to check out
		When I click the "Checkout Button"
		I should be redirected to the "Checkout Screen"
```

**selectors.json**
```
{
	"Checkout Button": ".checkout-button"
}
```

**screens.json**
```
{
	"Checkout Screen": "/checkout"
}
```

### End to End / Integration Test
End to End or Integration tests are similar to Feature tests, but they test an entire flow rather than just one feature. The same frameworks that can be used for Feature Tests (Cypress, Selenium, WebDriver, etc) can be used for E2E tests. You can also use abstractions such as Pickle for E2E tests. 

One big difference between a Feature and E2E test, is that the E2E uses a real backend (not mocked).

An example would be checking the full flow of a cart:
```
Feature: Cart Checkout
	Scenario: I should be able to add a cart item and checkout
		When I go to the "Login Screen"
		And I type "user@company.com" into the "Username Field"
		And I type "password" into the "Password Field"
		And I click "Add to Cart" on the "first Product Item"
		And I click the "Checkout Button"
		I should be redirected to the "Checkout Screen"
```

### Browser and Device Testing
Browser/Device testing is a subset of E2E testing. This is to ensure that functionality works as expected on different browsers and devices. This is especially important whenever dealing with device specific functionality (ex: different components on iPad/iPhone)

There are various "Virtual Browser" and "Virtual Device" providers which will help you with these tests. 

The main ones are:
- BrowserStack
- SauceLabs

They will allow you to test using WebDriver.io (and Pickle abstractions of WebDriver) on virtually any device/browser combination.

Browser/Device tests are written exactly like E2E tests, assuming there is a real backend. They are the closest to mirroring real world scenarios.

## The TDD/BDD Approach
### Your 1st Test
Starting a TDD approach may seem like an arduous task. However, even if you write just ` E2E test, you already get a huge return on investment. I've seen many situtations where a small update by a developer takes down an entire system. And you hear about it from the customer!

I recommend writing 1 test to start - testing the login to the application and seeing the front page being displayed. From there you can slowly and incrementally add on new tests.

The TDD/BDD approach suggests starting off with a set of requirements. You can write these requirements in plain English (or PickleJS) and slowly break down the process 

For example, if your Product Manager wants you to build a cart you may ask her to break down the specific behaviors:

```
Feature: Cart
	Scenario: I should be able to see a product catalog
		When I go to the "Product Selection Screen"
		Then I should see a "Product"
		And I should see a "Price" inside the "Product" containing "$10.00"
		And I should see a "Name" inside the "Product" containing "Pencil Sharpener"
		And I should see a "Description" inside the "Product" containing "A tool that sharpens pencils"
	
	Scenario: I should be able to add a product to my cart
		When I go to the "Product Selection Screen"
		And I click "Add to Cart" on the "Product"
		Then I should see "Product" inside the "Cart"
```

You write down every little piece of the functionality in this form, and boom! Your E2E tests and Feature Tests are practically done.

Next we just need to add Unit Tests for all the business logic.

```
import renderer  from  'react-test-renderer';
import ProductList from './ProductList';
import { mount } from 'enzyme';

test('Product List Component', () => {
	const items:[{
		name: "Pencil Sharpener",
		price: 10,
	}, {
		name: "Pencil",
		price: 1,
	}];
		
	it('should be able to list products', () => {
		const productList = renderer.create(<ProductList items={items}/>);
		expect(productList.toJSON()).toMatchSnapshot();
	});
	
	it('should render a total when an item is added to cart', () => {
		const productList = mount(<ProductList items={items}/>);
		productList.find('.product .addToCart').click();
		expect(productList.find('.cart
		expect(cart.toJSON()).toMatchSnapshot();
	});
});
```

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
eyJoaXN0b3J5IjpbODQzNjY3MDQyLC0xMTUzNzYxMzIxLDEzOT
U0MDE0OTcsODA1NTgzNTA3XX0=
-->