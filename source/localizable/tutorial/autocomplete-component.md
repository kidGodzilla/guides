As they search for a rental, users might also want to narrow their search
 to a specific city. Let's build a component that will let them search for
  properties within a city, and also suggest cities to them as they type.

To begin, let's generate our new component. We'll call this component 
`filter-listing`.

```shell
ember g component filter-listing
```
As before, this creates a Handlebars template 
(`app/templates/components/filter-listing.hbs`) and a JavaScript file 
(`app/components/filter-listing.js`).

The Handlebars template looks like this:

```app/templates/components/filter-listing.hbs
{{input action="search" value=filter key-up=(action 'autoComplete') class="light" placeholder="Search Cities"}}
<button {{action 'search'}} class="button light">Search</button>

<ul class="results">
  {{#each filteredList as |item|}}
      <li {{action 'choose' item.city}}>{{item.city}}</li>
  {{/each}}
</ul>
```
It contains an [`{{input}}`](../../templates/input-helpers) helper
that renders as a text field where the user can type a pattern to 
filter the list of cities used in a search. The `value` property of
the `input` will be bound to the `filter` property in our component.
The `key-up` property will be bound to the `autoComplete` action.

It also contains a button that is bound to the `search` action in our 
component.

Lastly, it contains an unordered list that displays the `city` property
of each item in the `filteredList` property in our component. Clicking 
the list item will fire the `choose` action with the `city` property of
the item as a parameter, which will then populate the `input` field with
the name of that `city`.

Here is what the component's JavaScript looks like:

```app/components/filter-listing.js
import Ember from 'ember';

export default Ember.Component.extend({
  filter: null,
  filteredList: null,
  actions: {
    autoComplete() {
      this.get('autoComplete')(this.get('filter'));
      Ember.$('.menu .results').show();
    },
    search() {
      this.get('search')(this.get('filter'));
    },
    choose(city) {
      this.set('filter', city);
      Ember.$('.menu .results').hide();
    }
  }
});
```

There's a property for each of the `filter` and `filteredList`, and 
actions as described above. What's interesting is that only the `choose` 
action is defined by the component. The actual logic of each of the 
`autoComplete` and `search` actions are pulled from the component's 
properties, which  means that those actions need to be [passed]
 (../../components/triggering-changes-with-actions/#toc_passing-the-action-to-the-component) 
 in by the calling object, a pattern known as _closure actions_.

To see how this works, change your `application.hbs` template to look like this:

```app/templates/application.hbs
<div class="container" {{action 'hideAutocomplete'}}>
    <div class="menu">
        {{#link-to 'index'}}
            <h1 class="left">
                <img src="http://emberjs.com/images/ember-logo.svg" alt="Ember Logo">
                <em>rentals</em>
            </h1>
        {{/link-to}}
        <div class="left links">
            {{#link-to 'about'}}
                About
            {{/link-to}}
            {{#link-to 'contact'}}
                Contact
            {{/link-to}}
        </div>
        <div class="right relative">
          {{filter-listing filteredList=filteredList
          autoComplete=(action 'autoComplete') search=(action 'search')}}
        </div>
    </div>
    <div class="body">
        {{outlet}}
    </div>
</div>
```
We've added the `filter-listing` component to our `application.hbs` template. We 
then pass in the functions and properties we want the `filter-listing` 
component to use, so that the application can define some of how it wants 
the component to behave, and so the component can use those specific 
functions and properties.

For this to work, we need to introduce a `controller` into our app. 
Generate a controller for the application by running the following:

```shell
ember g controller application
```

Now, define your new controller like so:

```app/controllers/application.js
import Ember from 'ember';

export default Ember.Controller.extend({
  filteredList: null,
  indexController: Ember.inject.controller('index'),
  actions: {
    autoComplete(param) {
      if (param !== '') {
        this.get('store').query('rental', { city: param }).then((result) => this.set('filteredList', result));
      } else {
        this.set('filteredList', null);
      }
    },
    search(param) {
      if (param !== '') {
        this.store.query('rental', { city: param }).then((result) => this.set('indexController.model', result));
      } else {
        this.get('store').findAll('rental').then((result) => this.set('indexController.model', result));
      }
    },
    hideAutocomplete() {
      Ember.$('.menu .results').hide();
    }
  }
});

```

As you can see, we define a property in the controller called 
`filteredList`, that is referenced from within the `autoComplete` action.
 When the user types in the text field in our component, this is the 
 action that is called. This action filters the `rental` data to look for 
 records in data that match what the user has typed thus far. When this 
 action is executed, the result of the query is placed in the 
 `filteredList` property, which is used to populate the autocomplete list 
 in the component.

We also define a `search` action here that is passed in to the component,
 and called when the search button is clicked. This is slightly different
  in that the result of the query is actually used to update the `model` 
  of the `index` route, and that changes the full rental listing on the 
  page.

For these actions to work, we need to modify the Mirage `config.js` file 
to look like this, so that it can respond to our queries.

```mirage/config.js
export default function() {
  this.get('/rentals', function(db, request) {
    let rentals = [{
        type: 'rentals',
        id: 1,
        attributes: {
          title: 'Grand Old Mansion',
          owner: 'Veruca Salt',
          city: 'San Francisco',
          type: 'Estate',
          bedrooms: 15,
          image: 'https://upload.wikimedia.org/wikipedia/commons/c/cb/Crane_estate_(5).jpg'
        }
      }, {
        type: 'rentals',
        id: 2,
        attributes: {
          title: 'Urban Living',
          owner: 'Mike Teavee',
          city: 'Seattle',
          type: 'Condo',
          bedrooms: 1,
          image: 'https://upload.wikimedia.org/wikipedia/commons/0/0e/Alfonso_13_Highrise_Tegucigalpa.jpg'
        }
      }, {
        type: 'rentals',
        id: 3,
        attributes: {
          title: 'Downtown Charm',
          owner: 'Violet Beauregarde',
          city: 'Portland',
          type: 'Apartment',
          bedrooms: 3,
          image: 'https://upload.wikimedia.org/wikipedia/commons/f/f7/Wheeldon_Apartment_Building_-_Portland_Oregon.jpg'
        }
      }];

    if(request.queryParams.city !== undefined) {
      let filteredRentals = rentals.filter(function(i) {
        return i.attributes.city.toLowerCase().indexOf(request.queryParams.city.toLowerCase()) !== -1;
      });
      return { data: filteredRentals };
    } else {
      return { data: rentals };
    }
  });
}
```

With these changes, users can search for properties in a given city, with
 a search field that provides suggestions as they type.


