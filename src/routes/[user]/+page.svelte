<script>
    import { onMount } from "svelte";
    import { fly, slide } from "svelte/transition";
    import { base } from "$app/paths";
    import { page } from "$app/stores";

    let intro = $state(false);

    let error = $state(false);

    let name = $state("");
    let trust = $state(0);
    let totalHours = $state("0h 0m");
    let dailyAverage = $state("0h 0m");

    onMount(function() {
        setTimeout(() => {intro = true;}, 100);
    })

    onMount(function() {
        document.addEventListener("keydown", function(event) {
            if (event.key == "Escape" && error) {
                window.location.href = base + "/";
            }
        })
    })

    onMount(async function() {
        let response = await fetch("https://hackatime.hackclub.com/api/v1/users/" + $page.params.user + "/stats");
        if (!response.ok) {
            error = true;
            name = "404";
        }
        else {
            let data = await response.json();
            name = data.data.username;
            trust = data.trust_factor.trust_value;
            totalHours = data.data.human_readable_total;
            dailyAverage = data.data.human_readable_daily_average;
        }
        
    })

</script>
<svelte:head>
    <title>{name} | Aftonsparv</title>
</svelte:head>

<style>
    .field {
        background-color: rgb(82, 95, 82);
        margin: 20px;
        border-radius: 30px;
        padding: 20px;
        position: relative;
    }
    .field span.text {
        background-color: rgb(140, 146, 140);
        padding: 10px;
        border-radius: 20px;
    }


    #row1 {
        width: 40vw;
    }
    #row2 {
        width: 47vw;
        height: 23vw;
        overflow-y: scroll;
        overflow-x: hidden;
    }

    div.normalTrust {
        background-color: rgb(42, 42, 83);
        margin-left: 30px;
        margin-right: 30px;
        padding: 5px;
        border-radius: 30px;
    }
    div.noTrust {
        background-color: rgb(83, 43, 42);
        margin-left: 30px;
        margin-right: 30px;
        padding: 5px;
        border-radius: 30px;
    }
    div.yesTrust {
        background-color: rgb(76, 156, 87);
        margin-left: 30px;
        margin-right: 30px;
        padding: 5px;
        border-radius: 30px;
    }
</style>

{#if intro}
    {#if !error}
    <h1 id="title" transition:slide>AFTONSPARV</h1>
    <h3 style="font-weight: 200; margin-bottom: 80px;">Hackatime Viewer</h3>
    <table>
        <tbody>
            <tr>
                <td>
                    <div class="field" id="row1">
                        <h6><span style="font-weight:500; font-size: 80px;" class="material-symbols-outlined" translate="no">account_circle</span></h6>
                        <h2>{name}</h2>
                        <h3 style="margin-bottom: 40px;"><span class="text">{$page.params.user}</span></h3>
                        {#if trust == 0}
                        <div class="normalTrust"><h3>Normal</h3></div>
                        {/if}
                        {#if trust < 0}
                        <div class="noTrust"><h3>Not Trusted</h3></div>
                        {/if}
                        {#if trust > 0}
                        <div class="yesTrust"><h3>Trusted</h3></div>
                        {/if}
                    </div>
                </td>
                <td>
                    <div class="field" id="row2">
                        <h3 style="font-family: Montserrat, Space Grotesk; font-weight: 800; font-size: 40px; margin-bottom: 2px;">{totalHours}</h3>
                        <h3>Daily average of {dailyAverage}</h3>
                    </div>
                </td>
            </tr>
        </tbody>
    </table>
    <button style="position: fixed; left: 10px; top: 20px;" onclick={() => {window.location.href = base + "/"}}><span class="material-symbols-outlined" translate="no">arrow_circle_left</span></button>
    {:else}
    <h1 style="font-size: 100px;">404</h1>
    <h2 style="margin-bottom: 100px">User could not be found. Maybe you entered the Slack ID incorrectly?</h2>
    
    <h4><button style="font-size: 15px;" onclick={() => {window.location.href = base}}>Return Home</button></h4>
    <p>You can also use <span style:font-family="Montserrat, Space Grotesk, Futura" style:font-weight = 700>ESC</span> Key</p>
    {/if}
{/if}