<script>
    import {onMount} from "svelte";
    import { fly } from "svelte/transition"

    let intro = $state(false);
    onMount(function() {
        setTimeout(() => {intro = true;}, 100);
    })
</script>
<style>
    #container {
        position: fixed;
        bottom: 0;
        height: 70px;
        left: 0;
        right: 0;
        background-color: rgb(43, 51, 43);
    } 

    #container div {
        position: relative;
    }

    #container div img { 
        position: absolute;
        transform: translate(-50%, 60%);
        left: 50%;
        max-width: 120px;
        bottom: 0;
    }
</style>

{#if intro}

<div id="container" transition:fly={{y:200}}>
    <div>
        <img src="/aftonsparv/images/afton.png" alt="Aftonsparv plushie" />
    </div>
</div>
{/if}