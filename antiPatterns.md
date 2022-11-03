React/Redux Anti-Patterns and Things to Avoid
=============================================
Here are some common mistakes you might make early on, also referred to as anti-patterns when developing in React and Redux.

Dispatching in render()
-----------------------
You should never dispatch() a Redux action from within a component's render() method, or any time outside of a user event (such as a click or hover). Doing so usually means you're trying to install business logic into the UI layer which is bad practice. It will also throw an error: "Warning: setState(...) can only update a mounted or mounting ...".

Example:
````js
render() {
    if (this.props.aValue) {
        this.props.dispatch(setBValue(true));
        return;
    }

    if (this.props.bValue) {
        return <div>B!</div>;
    }

    return <div></div>;
}
````
Here, the dispatch would update the Redux state which would cause a second call to render() on all components. It's also unnecessary. Either the props should be updated on the server side or you should return the `<div>B</div>` in the first `if()`.

*Solution:* Use only this.props or this.state in render() and never call dispatch() or do asychronous (e.g.: ajax) work.

Using deep values in state
--------------------------
React computes differences in state with a shallow compare. So { value: false} !== { value: true }. State changes trigger re-rendering and other event logic. React doesn't do a deep comparison: { value: { a: 1 } } == { value: { a: 2 } } so such a change won't trigger a state change. Additionally, it can't trigger state changes on arrays: [ 1, 2, 3 ] vs [ 1, 2, 3, 4 ].

Example:
````js
constructor(props) {
    super(props);
    this.state = { myObj: { name: 'Shawn' } };
}

changeSomething(e) {
    this.setState({ myObj: Object.assign({}, this.state.myObj, { name: e.target.value }) });
}
````
First, this is confusing as we have to duplicate the object and then modify a single key. Secondly, it *may not* trigger a state change since it's modifying a nested object. 

*Solution:* Flatten your state objects: this.state = { myObjName: 'Andrew' }. For arrays, the easiest fix is to include the length. Since the length key is flat, it will update the state: (e.g.: this.state = { items: [], itemsLength: 0 }).

Use propTypes
-------------
For components, utilize propTypes when you're not working with swagger model or redux properties to ensure consumers abide by your requirements.
````js
MyComponent.PropTypes = {
    workorderId: React.PropTypes.number.isRequired,
    rates: React.PropTypes.array,
    onChange: React.PropTypes.func,
};

MyComponents.defaultProps = {
    workorderId: 0,
    rates: [],
};
````
Here we've stated workorderId must be numeric (defaults to 0) and that there should be an onChange callback. Using propTypes guarantees your component will get variables of the types you expect. It means less typing things like "if (typeof this.props.onChange == 'function') {" and keeps your code cleaner and easier to maintain and test. If you ever get an unexpected value, React will throw a clean error with line numbers so you can correct your input.

When passing props from unverified sources (like server endpoints), make sure you sanitize it to match your propTypes:
````js
{
    workorderId: +server.workorderId, // forces a numeric value
    name: '' + server.name, // forces a string
    isRequired: !!server.required, // casts as a boolean true/false
    rates: server.rates || [], // if server.rates is null, false, '', or 0 then use an empty array
    checked: server.checked ? true : false, // force a boolean value
}
````

Always clone objects you intend to modify
-----------------------------------------
This is the *most* important tip: unlike PHP, in javascript, all objects are pointers (like the & operator in php) and changing them will change all other references. Updating the state in React/Redux is very bad and will introduce loads of problems that aren't easy to identify.

Example:
````js
onChange(e) {
    // very bad, you're updating the this.state object!
    this.state.myObj.name = e.target.value;
}
````

*Solution:*
````js
onChange(e) {
    this.setState({ myObj: { ...myObj, name: e.target.value } });
}
````
Using the ES6 spread operator ... is like array_merge in PHP. We first pass an empty object so that becomes the "source" object which gets all the changes. The second or later objects passed will strictly be copied and NOT modified. This might be more clear to understand:

Example:
````js
let a = { name: 'Andrew' };
let b = a;
b.name = 'Bill';
console.log(a.name); // Bill
````

*Solution:*
````js
let a = { name: 'Andrew' };
let b = { ...a };
b.name = 'Bill';
console.log(a.name); // Andrew
````

Arrays are also objects
-----------------------
Similar to the last pattern, arrays are also objects in javascript. Modifying them also modifies the pointer to the object. The solution is similar to Object.assign except we use [].concat.

Example:
````js
let a = [1, 2, 3];
let b = a;
b.push(4);
console.log(a); // 1, 2, 3, 4
````

*Solution:*
````js
let a = [1, 2, 3];
let b = [].concat(4);
console.log(a.name); // 1, 2, 3
````

Be consistent in naming
-----------------------
In React/Redux world, try to stay consistent and use underscore notation: first_name vs. firstName.

Use global components (or add them) whenever possible
-----------------------------------------------------
As a rule of thumb, whenever you're writing CSS or significant markup in a smart/connected/Redux component, you should create a global component with a demo instead. CSS should never be applied to Redux components. This allows us optimal code reuse and UX/front-end review/control.

Commonly missed global components include:
- Button
- FormInput
- Modal
- Alert
- Currency
- DateTime
- Email
- Address1, Address2
