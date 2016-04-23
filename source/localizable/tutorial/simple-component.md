As a user looks through our list of rentals, they may want to have some interactive options to help them make a decision.
Let's add the ability to hide and show an image for each rental.
To do this, we'll use a component.

Let's generate a `rental-listing` component that will manage the behavior for each of our rentals.
A dash is required in every component name to avoid conflicting with a possible HTML element.
So `rental-listing` is acceptable but `rental` would not be.

```shell
ember g component rental-listing
```

Ember CLI will then generate a handful of files for our component:


```shell
installing component
  create app/components/rental-listing.js
  create app/templates/components/rental-listing.hbs
installing component-test
  create tests/integration/components/rental-listing-test.js
```

A component consists of two parts:
a Handlebars template that defines how it will look (`app/templates/components/rental-listing.hbs`)
and a JavaScript source file (`app/components/rental-listing.js`) that defines how it will behave.

Our new `rental-listing` component will manage how a user sees and interacts with a rental.
To start, let's move the rental display details for a single rental from the `index.hbs` template
into `rental-listing.hbs`:

```app/templates/components/rental-listing.hbs
<article class="listing">
    <a>
    <img src="{{rental.image}}" alt="">
    <small>View Larger</small>
    </a>
    <h3>{{rental.title}}</h3>
    <div class="detail">
        <span>Owner:</span> {{rental.owner}}
    </div>
    <div class="detail">
        <span>Type:</span> {{rental.type}}
    </div>
    <div class="detail">
        <span>Location:</span> {{rental.city}}
    </div>
    <div class="detail">
        <span>Number of bedrooms:</span> {{rental.bedrooms}}
    </div>
</article>
```

In our `index.hbs` template, let's replace the old HTML markup within our `{{#each}}` loop
with our new `rental-listing` component:

```app/templates/index.hbs
<div class="jumbo">
    <img class="right" src="http://emberjs.com/images/blog/2016-02/lts-tomster.png" alt="Tomster">
    <h2>Welcome!</h2>
    <p>
        We hope you find exactly what you're looking for in a place to stay.
        <br>Browse our listings, or use the search box above to narrow your search.
    </p>
    {{#link-to 'about' class="button"}}
        About Us
    {{/link-to}}
</div>

{{#each model as |rentalUnit|}}
  {{rental-listing rental=rentalUnit}}
{{/each}}
```
Here we invoke the `rental-listing` component by name,
and assign each `rentalUnit` as the `rental` attribute of the component.

## Hiding and Showing an Image

Now we can add functionality that will enlarge the image of a rental when requested by the user.

Let's use the `{{#if}}` helper to add the class `wide` to our current link only when `wide` is set to true.

```app/templates/components/rental-listing.hbs
<article class="listing">
    <a class="{{if wide "wide"}}">
    <img src="{{rental.image}}" alt="">
    <small>View Larger</small>
    </a>
    <h3>{{rental.title}}</h3>
    <div class="detail">
        <span>Owner:</span> {{rental.owner}}
    </div>
    <div class="detail">
        <span>Type:</span> {{rental.type}}
    </div>
    <div class="detail">
        <span>Location:</span> {{rental.city}}
    </div>
    <div class="detail">
        <span>Number of bedrooms:</span> {{rental.bedrooms}}
    </div>
</article>
```

The value of `wide` comes from our component's Javascript file, in this case `rental-listing.js`.
Since we do not want the image to be showing at first, we will set the property to start as `false`:

```app/components/rental-listing.js
export default Ember.Component.extend({
  wide: false
});
```

To make it where clicking on the image makes it larger,
we will need to add an action that changes the value of `wide` to `true`.
Let's call this action `enlargeImage`

```app/templates/components/rental-listing.hbs
<article class="listing">
    <a {{action 'enlargeImage'}} class="{{if wide "wide"}}">
    <img src="{{rental.image}}" alt="">
    <small>View Larger</small>
    </a>
    <h3>{{rental.title}}</h3>
    <div class="detail">
        <span>Owner:</span> {{rental.owner}}
    </div>
    <div class="detail">
        <span>Type:</span> {{rental.type}}
    </div>
    <div class="detail">
        <span>Location:</span> {{rental.city}}
    </div>
    <div class="detail">
        <span>Number of bedrooms:</span> {{rental.bedrooms}}
    </div>
</article>
```

Clicking this button will send the action to the component.
Ember will then go into the `actions` hash and call the `enlargeImage` function.
Let's create the `enlargeImage` function and toggle `wide` on our component:

```app/components/rental-listing.js
export default Ember.Component.extend({
  isImageShowing: false,
  actions: {
    enlargeImage() {
      this.toggleProperty('wide');
    }
  }
});
```

Now when we click the button in our browser, our image becomes larger. When we click it again, it shrinks back to its original size.

