<!--
    @component
    This component can be used to load an image lazily, like this:
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
-->

<script lang="ts">
    import { createEventDispatcher } from 'svelte'

	const dispatch = createEventDispatcher()
    interface RestProps {
        [key: string]: string | number | boolean
    }
    export let src: string
    export let alt: string
    export let props: RestProps
    /**
     * A string like "100px" defining how much earlier it should be loaded
     */
    export let rootMargin: string = undefined
    $: boundSrc = ''
    let element: HTMLImageElement = null
    
    $: if (element instanceof HTMLImageElement) {
        const observer = new IntersectionObserver(async (elements) => {
            for (const observedElem of elements) {
                if (observedElem.isIntersecting) {
                    observer.unobserve(element)
                    boundSrc = src
                    dispatch('load', element)
                }
            }
        }, {
            rootMargin
        })
        observer.observe(element)
    }
</script>

<img bind:this={element} src={boundSrc} alt={alt} {...props} />