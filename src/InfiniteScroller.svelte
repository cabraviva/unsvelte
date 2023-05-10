<!--
    @component
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
-->
<script lang="ts">
    import { onMount } from 'svelte'

    interface PropsObject {
        [key: string]: any
    }
    interface Options {
        /**
         * Async function that returns an array of objects, which contain props that
         * will be passed into the displayComponent. The number of items returned
         * for a page should be kept fairly low, as every time the user reaches the bottom
         * one page get's loaded. The first page index will be page 0
         * @param page Current page to load data for
         * @example 
         * ```js
         * async (page) => {
         *      return [{
         *          prop1: 'value',
         *          page
         *      }, {
         *          prop1: 'value2',
         *          page
         *      }]
         * }
         * ```
        */
        getData(page: number): Promise<PropsObject[]>,
        /**
         * Async function that returns number of pages available
         * @example
         * ```js
         * async () => {
         *      const res = await fetch('/api/number-of-pages')
         *      return await res.json()
         * }
         * ```
         */
        pageCount(): Promise<number>,
        
        /**
         * A function that can be used to style / set layout for the scroller.
         * It will be called with the wrapper element.
         * @param element Element that will hold all items
         * @example
         * ```js
         * (element) => {
         *      element.style.display = 'flex'
         *      element.style.justifyContent = 'center'
         * }
         * ```
         */
        initLayout(element: HTMLDivElement): void

        /**
         * The component that will be invoked for every item that should be displayed
        */
        displayComponent: any,
        /**
         * Here you can optionally provide a number of loaded items,
         * at which the first `overflowDelete` items will be deleted.
         * If not set, nothing will be deleted
         * NOTE: They will never come back!
         */
        overflow?: number
        /**
         * Number of items to delete when overflow is reached
         * @default 100
        */
        overflowDelete?: number
    }
    export let data: Options

    function overflowCheck () {
        if (data.overflow && data.overflow >= counter) {
            Array.from(document.body.querySelectorAll('*')).slice(0, data.overflowDelete || 100).forEach(e => e.remove())
        }
    }
    
    let wrapper: HTMLDivElement = null
    let lastItemIndicator: HTMLDivElement = null
    let counter = 0

    const componentId = btoa(crypto.getRandomValues(new Uint8Array(30)).join('') + Date.now()).replace(/=/g, '')

    async function loadPage (pageIndex: number) {
        const items = await data.getData(pageIndex)
        for (const item of items) {
            const div = document.createElement('div')
            div.id = `unsvelte-${componentId}-${counter}`
            wrapper.appendChild(div)
            new data.displayComponent({
                target: wrapper.querySelector(`#unsvelte-${componentId}-${counter}`),
                props: item || {}
            })
            counter += 1
        }
    }

    onMount(async () => {
        const pageCount = await data.pageCount()
        if (pageCount === 0) return
        let pageToLoadNext = 1
        loadPage(0)
        if (wrapper instanceof HTMLDivElement) {
            data.initLayout(wrapper)
            const observer = new IntersectionObserver((observedElements) => {
                if (observedElements.length === 0) return
                const observedElement = observedElements[0]
                if (observedElement.isIntersecting) {
                    if (pageCount >= pageToLoadNext) {
                        loadPage(pageToLoadNext)
                        pageToLoadNext += 1
                        overflowCheck()
                    } else {
                        // No more pages available, unobserve
                        observer.unobserve(lastItemIndicator)
                    }
                }
            }, {
                rootMargin: `${window.screen.availHeight}px`
            })
            observer.observe(lastItemIndicator)
        }
    })
</script>

<div bind:this={wrapper}></div>
<div bind:this={lastItemIndicator}></div>

<!--
    Usage:
    <InfiniteScroller data={{
        async getData (page: number) {
            return [{
                prop1: 'HelloWorld!'
            }]
        },
        async pageCount () {
            return 1
        },
        initLayout (element: HTMLDivElement) {
            element.style.display = 'flex'
            element.style.justifyContent = 'center'
        }
        displayComponent: MyComponent
    }} />
-->