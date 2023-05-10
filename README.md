# unsvelte
 Lazy loading components for svelte 🦥
 
# Components
 There are three components that come with unsvelte, which you can all import separately, so your bundle stays tiny:
 
## 1. Island
 The simplest and yet most powerful component of those is the Island. It's basically a dummy component that will be replaced with your actual component when visible.
 To use it, first create a separate component file, for example called `MyComponent.svelte`, then you could use the Island like this:
 ```svelte
<Island content={() => import('./MyComponent.svelte')} />
 ```
 Yep, it's that easy.
 If you want to load your component a bit earlier, maybe because the user has a bad connection (which you can btw find out using [webperf](https://npmjs.com/package/webper)), you can change the rootMargin:
  ```svelte
<Island content={() => import('./MyComponent.svelte')} rootMargin="500px" />
 ```

## 2. LazyImg
 Still pretty simple. If you want to lazy load an image (maybe because you e building an image gallery), you can just use this component.
 It can be used to load an image lazily, like this:
 ```svelte
<LazyImg src="/my.jpg" alt="An image" />
```
 The actual source will first be defined when the image is in the users viewport.

 If you have additional props you want to pass down to the image, you can use the `props` property
 ```svelte
<LazyImg src="/my.jpg" alt="An image" props={{
    height: '10px',
    width: '10px'
}} />
 ```

 Optionally, you can also set how much earlier the image should be loaded:
 ```svelte
<LazyImg src="/my.jpg" alt="An image" rootMargin="100px" /> 
 ```

## 3. InfiniteScroller
 This one is rather complex, but still simplifies it a lot for you.
 You can use this component to create a infinite scroll.
 This expects that every item you want to render is based on the same
 component, and will be rendered differently based on props.
 It also expects that you have an api which allows you to fetch a page at a time.

 To use this component, just initiate it with a data object containing this:
 ```svelte
<InfiniteScroller data={{
    async getData (page: number) {
        // Fetch items on a specific page and
        // return props for the component here!
        return [{
            prop1: 'HelloWorld!'
        }]
    },
    async pageCount () {
        // Fetch the page count here
        return 1
    },
    initLayout (element: HTMLDivElement) {
        // You can use this to define your layout
        element.style.display = 'flex'
        element.style.justifyContent = 'center'
    }
    // This should be the item component
    displayComponent: MyComponent
}} />
 ```

 If you are expecting to deal with a really huge amount of items,
 you can define a overflow to automatically delete first items
 when new ones get initialized.
 NOTE: Your old items will never come back!

 ```svelte
<InfiniteScroller data={{
    ...yourOtherStuff,
    // Maximum amount of items
    overflow: 1000,
    // Number of items to delete if overflow was hit
    overflowDelete: 100
}} />
 ```