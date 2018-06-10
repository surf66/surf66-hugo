+++
author = "Tom Pennington"
date = "2018-06-10T10:54:24+02:00"
title = "Unit Testing React with Jest"
tags = ["Testing","React","Jest"]
+++

I've been writing React components and applications for a long time now and everytime I do, I always use the same unit testing setup and tools - [Karma](https://karma-runner.github.io/2.0/index.html) and [Jasmine](https://jasmine.github.io/). There's really nothing wrong with these tools, they've been around for a very long time, are well maintained and they provide everything you need to properly test your JavaScript. However, I wanted to see what all the fuss around Jest was. I've had a play this weekend and the below is what I thought.

## Setup & Configuration

Within minutes of using Jest you already see the first benefit, the ease of configuration. Just `npm install jest` add an `npm test` command into your `package.json` and you're done! With Karma thers a rather large `karma.conf.js` file you need first and theres also some fiddly Karma related stuff you need in your webpack config (if you're using webpack) and want your tests to understand es6 or 7.

## Snapshot Testing
This is what seems to be the main hype around Jest. At first I really didn't know what this was and how it worked, however on the Jest website theres a really helpful talk on snapshot tests which I'd definetly recommend.

> https://www.youtube.com/watch?v=HAuXJVI_bUs

Essentially, a snapshot test will capture the entire output of a function you are testing and compare the actual output with the snapshot when the test runs. Snapshots are stored along side your tests and are committed into source control.

### Benefits
Firstly, you actually end up covering more of your code with snapshot tests. Take the following example of a simple React component's render method.

```
render() {
  return (
    <div>
      <ul>
        {this.props.vehicles.map((vehicle, index) => {
          <li key={index}>{vehicle.make} {vehicle.model}</li>
        })}
      </ul>
      <p>Look at my cars</p>
      <img src="car-image.png" />
    </div>
  )
}
```

Here we are rendering a list of vehicles, simple. There's also some copy and an image below. Usually when I test a component like this, I would have a couple of different tests. One for the list of vehicles - check they render properly and what happens when there are none etc. I would also probably pick out the image and check that's renderd properly too. I havent tested the `<p>` at this point as I see it as unimportant, it's just copy. I don't care if theres a spelling error. My product owner or designer might but that's for another day.

Well, with a snapshot test you kind of get this `<p>` tested for free! As I said before, a snapshot test captures what the output of a function should look like.

```
test('should match snapshot', () => {
  const tree = shallow(<VehicleList vehicles={vehicles} />);

  expect(tree).toMatchSnapshot();
});
```

A snapshot test really looks that simple. Our unit test now verifies the entire output of the render method as opposed to me just hand picking the stuff I wanted to test.

#### Updating Snapshots
Let's say you check in, another dev comes along and adds in more copy, updates an image and your snapshot test fails. If you are happy your code is working, the snapshot just needs to be updated. You can simply run `npm run test -- --u` and it will update your snapshot and all is well.

This brings me nicely onto the next main benefit I found using Jest. It's quicker to test and easier to fix/update failing tests. Take the example below of an old approach to testing using Jasmine.

```
it('should return object', {
  const component = shallow(<MyComponent />);

  let response = component.instance()._myCallback();

  expect(response).toEqual([
    {"make": "audi","model": "a1"},
    {"make": "audi","model": "a4"}
  ]);
});
```

The above test ensures our method returns an array of vehicles. Say we change our code so that our object now contains `colour` and then in another few days we add a `fuelType` field. We constantly need to keep coming back into this test to update the expected output. With a snapshot test, you don't! Jest will see that your code no longer matches the snapshot and you will either update the snapshot or fix your code. It's that simple.

#### Stateful Components
You can also have multiple snapshots for the same component. A lot of the time our components behave differently and render different output based on state or props. Jest let's you set the state of a component before hand and run a snapshot test so you can cover each different state of your components.

## Mocking
At this point I'm only 2-3 hours into my time with Jest - but mocking looks weird. From the Jest docs, it seems you have to create a mock class for every class in your code that you want to mock out in unit tests. These mocks live in a directory named `_mocks_` along side your class.

I say this looks weird - it's not it's just new to me. It's just in Jasmine it's so so easy to mock classes and methods out by something like:

```
spyOn(MyClass.prototype, 'myMethod').and.returnValue({"key": "value"});
```

I couldn't see anything like the above in the Jest docs (I may be wrong). Everything was leading me down the path of creating a fake version of my class. This is ok, it's very explicit and it cleans up your tests. You simply import your mock class and Jest just knows to use your mock instead of the real thing.

## That's as far as I got

To learn all this I created a very basic React application which listed vehicles and allowed you to add some more. All the components in it are tested using Jest and you can find it on my GitHub, I hope this comes in handy!

> https://github.com/surf66/jest-fiddle



