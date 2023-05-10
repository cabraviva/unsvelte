<!--
    @component
    This component allows you to lazy load a different component.
    Just pass in a **content** function that returns an import promise.
    
    Here's an example:   
    ```svelte
    <Island content={() => import('./MyComponent.svelte')} /> 
    ```

    This will load the MyComponent as soon as it's visible.

    Optionally, you can also set how much earlier the component should be loaded:
    ```svelte
    <Island content={() => import('./MyComponent.svelte')} rootMargin="100px" /> 
    ```
-->

<script lang="ts">
    export let content: Function
    /**
     * A string like "100px" defining how much earlier it should be loaded
     */
    export let rootMargin: string = undefined

    $: element = null
    $: component = null

    $: if (element instanceof HTMLDivElement) {
        const observer = new IntersectionObserver(async (elements) => {
            for (const observedElem of elements) {
                if (observedElem.isIntersecting) {
                    observer.unobserve(element)
                    component = (await content()).default
                }
            }
        }, {
            rootMargin
        })
        observer.observe(element)
    }
</script>

<div bind:this={element}>
    {#if component}
        <svelte:component this={component} />
    {/if}
</div>
