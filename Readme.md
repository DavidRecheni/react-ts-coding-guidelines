# React TS Best practices and standards

# Code conventions

## No unused code

To keep the codebase light, we should remove every bit of unused code. That means code that was commented out, unused variables or components that because of code changes are not used anymore.

Don’t leave placeholder for future reference unless you are going to start with that other feature right away, otherwise it’ll probably be forgotten and stay there forever.

## TS/TSX Only

No JS components under any circumstance.

## Translation keys

When adding new code always use a translation key for the text that the user will see. They should follow the general pattern of `page.key` or `generic.key` in case is a generic word like "cancel". 

Translation strings should not be passed down as props but should instead be translated at the first point they enter a render function. 

```jsx
const ParentComponent = ({text}) => {
	return <p>{text}</p>
}

<ParentComponent text={t('page.key')}/>
```

## Hooks

Regardless of size and complexity hooks are all treated the same:
- If it's used only by **one** component, it should be stored in the folder of that component. 
- If it's used by **multiple** components within the **same** module, it should be stored in the *hooks* folder of the module. 
- If it's used by **multiple** components in **different** modules, it should be stored in the *hooks* folder of the `shared` module.
  
## Naming conventions

### Abbreviations: 
Treat abbreviations like acronyms in names as whole words:
```jsx
// good ✅ 
 loadHttpUrl
```

```jsx
// bad ❌
 loadHTTPURL
```
 
### Variables:
To properly name the elements we create is a key practice to keep the platform maintainable. The community has adopted the ***camelCase*** convention to name them. Also make sure the name you give to the variables are representative of what they’ll hold:

```jsx
// good ✅ 
 readBooks.forEach((book) => book.page)
```

```jsx
// bad ❌
 res.forEach((e) => e.page)
```

Any developer should be able to understand what’s going on on a first glance without having to browse through different components to check what the assigned value will be.

### Test IDs

Using ***kebab-case*** makes them easy to identify and differentiate from variable names

```jsx
<div test-id="this-is-a-test-id" />
```

## Comments

In general your code should be modularized and self-explanatory and not require comments. If necessary, we should comment **what** is being done not **how** it’s being done.

## Typescript

Do not use `@ts-ignore` unless you have a really *really* **really** good reason to. Examples of really good reasons: 

- Bugs in Typescript itself that are reported by the Typescript community 👥
- Issues that no one on the team can figure out 😁

### Interfaces and types

Storing them follows the same conventions as the *hooks*:

- If the type is only used in **one** file, it should be included in **that file** above the component interface.

- If it’s used by **multiple** files in a module, it should be included in the folder *types* of that module.

- If it’s used by **multiple modules**, it should be included in the folder *types* of the *shared* module

We should always use ***interfaces*** for objects and keep ***types*** *only* for functions. Since the latest is just an alias not does not offer the possibility to be extended.

To make them easier to identify, ***interfaces*** should start with an *“I”* and be followed by the name of the object it’s referencing, ie: `IPerson`. And ***types*** should start with *"T"* followed by the name of the function it's referencing, ie: `TExtractElement`

The component props are always named `IProps` and included in the same file before the component is declare but after the imports. As that interface will be unique for that component it shouldn't be in a separate file.

```jsx
import something from 'library'

interface IProps {
  someProp: any
}
const Component = (props : IProps) => {
  return <></>
}
```

### Defining types

If the type takes for one of its props only specific strings, define them. The use of a *string* primitive is incorrect in this case.  
The use the keyword *any* should be avoided whenever possible. The one valid use for it only when it’s **not possible** to know in advance what that prop will contain. 

Optional properties should always be placed at the bottom of any type / interface. 

```jsx

// good ✅ 
interface IProps {
  fieldName: string
  someProp?: 'this' | 'that'
}

// bad ❌
interface IProps {
  someProp?: string
  fieldName: any
}
```



# System architecture

## Data handling

**Every** API request must be stored inside a `service` file.  

If a component needs data that comes from the backend, it should be fetched from within that component. If necessary it should take an *id* as prop and fetch it using useQuery, with a query key defined for that API call as parameter.

The cache keys will be stored on every module in the *constants* folder. Inside of a `QueryStrings` file.

The function used to fetch the data has to be imported from the *service* file from the module we are working on. 

## Folder Structure

```jsx
📁 modules
-- 📁 GeneralModuleName
---- 📁 constants
------ 🔡 QueryStrings.ts
---- 📁 utils
---- 📁 services
---- 📁 hooks
---- 📁 types
---- 📁 subModules
------ 📁 SpecificSectionName
-------- 🔡 SpecificSectionNameContainer.tsx
-------- 📁 components
---------- 📁 SpecificComponentName
------------ 🔡 SpecificComponentName.tsx
------------ 🧪 SpecificComponentName.spec.tsx
------------ 📁 InnerComponentName
------------ 🔡 utils.ts
```

In general, `utils` within components should not be reusable between other specific components. If the util is reused it should be moved to a common parent’s `utils` file. 

A component should not have more than 5 inner components in their folder. Otherwise a further `components` folder should be used within the component.

## Components

### Modularization

Try to ensure every component achieves *one* thing. Some examples of things that should be their own component and files are…

- A button that deletes a tag
- An input that adds a tag
- A hook that organizes the tag
- A container that renders all of the tags

This also includes detaching the UI from the component logic whenever possible, because it’s likely to change often and we don’t want to refactor every component every time it happens. For example if we have this button:

```jsx
<Icon onClick={doSomething} />
```

If instead of having an icon we want to have a text “Add”, we would have to refactor it.

But if the component structure is:

```jsx
<Button onClick={doSomething}>
  <Icon/>
</Button>
```

We can easily change it to:

```jsx
<Button onClick={doSomething}>
  <p> Add </p>
</Button>
```

### DRY (Don’t Repeat Yourself)

Anytime you see duplicated code you should think to see if you can simplify it. For example, if you call the same component multiple times with different props, consider extracting the props to an array/hook and mapping over them in the UI.


# Testing

## General considerations

Whenever new functionality is added, the PR should include testing for that functionality. Cypress tests are stored in the `cypress` folder while unit tests are stored inside the components folder.

Before submitting a PR, if it's not done automatically, remember to run all tests to make sure your functionality didn’t break something else and to check that the code coverage is above the minimum accepted by the team. 

## Unit tests

If you are developing a highly complex function (sorting algorithm / validation) you should ideally write unit tests and ensure that they pass every time you make changes to it. 

Unit tests files must be named `fileName.spec.tsx`