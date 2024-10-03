<script>
    import Background from "$lib/Background.svelte";
    import MainMenu from "$lib/MainMenu.svelte";
    import WorldMenu from "$lib/WorldMenu.svelte";
    import Kaplay from "$lib/Kaplay.svelte";
    import Instruction from "$lib/Instruction.svelte";

    // import { facts } from '$lib/facts'

    import { browser } from '$app/environment'

    let isMenu = true
    let page = 0
    let world, l, lWorld
    let mL = {
        w1:null,
        w2:null,
        w3:null
    }

    $: outerWidth = 0
	$: innerWidth = 0
	$: outerHeight = 0
	$: innerHeight = 0

    let ori
    let ls

    $: if (innerWidth > innerHeight) {
        ori = "l"
    } else if (innerWidth < innerHeight) {
        ori = "p"
    }

    if (browser) {
        ls = window.localStorage.getItem("kply-lvl") //get last highest playable level
        if (!ls) {
            console.log("first time")
            mL.w1=1
            mL.w2=1
            mL.w3=1
            window.localStorage.setItem("kply-lvl", JSON.stringify(mL))
        } else {
            mL = JSON.parse(ls)
            // console.log(mL, "max last level")
        }
    }

</script>

<svelte:head>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Jua&display=swap" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Nunito:ital,wght@0,200..1000;1,200..1000&display=swap" rel="stylesheet">
</svelte:head>

<svelte:window bind:innerWidth bind:outerWidth bind:innerHeight bind:outerHeight />

<section style:display={isMenu ? "flex" : "none"}>
    <Background>
    {#if page == 0}
        <MainMenu 
            bind:page
        />
    {:else if page == 1}
        <WorldMenu 
            bind:page 
            bind:world
        />
    {:else if page == 2}
        <Instruction
            bind:page 
            bind:world
            bind:isMenu
        />
    {/if}
    </Background>
</section>

{#if !isMenu}
<Kaplay bind:isMenu bind:world {l}/>
{/if}

<style>
    :global(button) {
        font-family: 'Nunito', sans-serif;
    }
    :global(body) {
        /* disable scroll */
        overflow: hidden;
        margin:0;
    }
    section {
        width:100%;
        height:100vh;
        position:absolute;
        display:flex;
        justify-content: center;
        align-items: center;
        /* background-color: cornflowerblue; */
        gap:0.5rem;
    }
</style>
