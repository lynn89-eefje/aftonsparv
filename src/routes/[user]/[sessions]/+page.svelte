<script>
    import { onMount } from "svelte";
    import { fly, slide } from "svelte/transition";
    import { base } from "$app/paths";
    import { page } from "$app/stores";

    let errorMessage = $state([
        "404",
        "User could not be found. Maybe you entered the Slack ID incorrectly?"
    ])

    let intro = $state(false);

    let error = $state(false);

    let name = $state("Loading");
    let trust = $state(0);
    let totalUserSecs = 0;

    let addedSecs = $state(0);
    let langs = $state([]);
    let displayHours = $state("0h 0m");

    let sessions = $state([]);
    let displaySessions = $state("");

    $effect(() => {
        if (sessions.length > 1) {
            let text = "Filtered by sessions: "
            for (let i = 0; i < sessions.length-1; i++) {
                text += sessions[i] + ", "
            }
            text += sessions[sessions.length-1];
            displaySessions = text;
        }
        else {
            displaySessions = "Filtered by session: " + sessions[0];
        }
    })

    onMount(async () => {
        if ($page.params.sessions.indexOf("[x]") > 0) {
            let processText = "";
            for (let i = 0; i < $page.params.sessions.length; i++) {
                if (processText.indexOf("[x]") > 0) {
                    sessions.push(processText.substring(0, processText.length-3));
                    i--;
                    processText = ""; 
                }
                else {
                    processText += $page.params.sessions.substring(i, i+1);
                }
            }
            if (processText.length > 0) {
                sessions.push(processText.substring(0, processText.length-3));
            }
        }
        else {
            sessions = [$page.params.sessions];
        }
        console.log(sessions);
        let response = await fetch("https://hackatime.hackclub.com/api/v1/users/" + $page.params.user + "/stats");
        if (!response.ok) {
            error = true;
            name = "404";
        }
        else {
            let data = await response.json();
            name = data.data.username;
            trust = data.trust_factor.trust_value;
            totalUserSecs = data.data.total_seconds;

            for (let i in sessions) {
                //console.log(sessions[i]);
                response = await fetch("https://hackatime.hackclub.com/api/v1/users/" + $page.params.user + "/stats?filter_by_project=" + sessions[i]);
                data = await response.json();
                //console.log(data.data.total_seconds);
                if (data.data.total_seconds < totalUserSecs && data.data.total_seconds != 0) {
                    addedSecs += data.data.total_seconds;
                }
                else {
                    error = true;
                    name = "404";
                    errorMessage[1] = "'" + sessions[i] + "' " + " session was not found. Did you mispell the session name?";
                    return 0;
                } 
            }
            if (addedSecs > totalUserSecs) {
                console.log("ERROR: Added secs is greater than total user secs", addedSecs, totalUserSecs);
                errorMessage[0] = "ERROR"
                errorMessage[1] = "Safety conditional on line 86 is true";
                error = true;
                return 0;
            }
            console.log(addedSecs);
            for (let i in sessions) {
                response = await fetch("https://hackatime.hackclub.com/api/v1/users/" + $page.params.user + "/stats?filter_by_project=" + sessions[i]);
                data = await response.json();
                for (let x in data.data.languages) {
                    let check = false;
                    if (langs.length > 0) {
                        for (let k in langs) {
                            if (langs[k].name == data.data.languages[x].name) {
                                //console.log("Conditional on 104 is true", data.data.languages[x].name, langs[k].name)
                                langs[k].total_seconds += data.data.languages[x].total_seconds;
                                check = true;
                                break;
                            }
                        }
                    }
                    if (!check) {
                        //console.log(data.data.languages[x].name, data.data.languages[x].total_seconds);
                        langs.push({
                            "name": data.data.languages[x].name,
                            "total_seconds": data.data.languages[x].total_seconds,
                            "sessions": [sessions[i]]
                        })
                    }
                }
                console.log(langs);
            }
        
            //console.log(langs);
            for (let x in langs) {
                langs[x].percent = Math.round((langs[x].total_seconds/addedSecs)*100);
                let mins = Math.floor(langs[x].total_seconds/60);
                let hours = Math.floor(mins/60);
                let secs = langs[x].total_seconds - (mins*60);
                mins = mins - (hours*60);
                langs[x].text = hours + "h " + mins + "m " + secs + "s";
            }
            if (true) {
                let mins = Math.floor(addedSecs/60);
                let secs = addedSecs - (mins*60);
                let hours = Math.floor(mins/60);
                mins -= hours*60;
                displayHours = hours + "h " + mins + "m " + secs + "s";
            }

        }
    })

    onMount(function() {
        setTimeout(() => {intro = true;}, 100);
    })

    onMount(function() {
        document.addEventListener("keydown", function(event) {
            if (event.key == "Escape") {
                window.location.href = base + "/";
            }
        })
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
        width: 29vw;
    }
    #row2 {
        width: 58vw;
        height: 350px;
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

    #langContainer {
        display: grid;
        grid-template-columns: auto auto;
        justify-content: center;
    }

    .lang {
        background-color: rgb(121, 134, 121);
        margin: 10px;
        border-radius: 30px;
        width: 27vw;

    }
</style>
<!--
    span.text {
        background-color: rgb(56, 71, 56);
        padding: 5px;
    }
-->

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
                        <h4>{displaySessions}</h4>
                        <h3 style="font-family: Montserrat, Space Grotesk; font-weight: 800; font-size: 40px; margin-bottom: 2px;">{displayHours}</h3>
                        <h2 style="margin-bottom: 1px">By Language</h2>
                        <p>({langs.length})</p>
                        <div id="langContainer">
                            {#each langs as x}
                                <div class="lang">
                                    <h4 style="font-size: 25px;">{x.name}</h4>
                                    <h5 style="font-size: 20px;">{x.text} <!--<span class="text" style="padding: 5px; margin-left: 5px;">{x.percent}%</span>--></h5>
                                </div>
                            {/each}
                        </div>
                    </div>
                </td>
            </tr>
        </tbody>
    </table>
    <button style="position: fixed; left: 10px; top: 20px;" onclick={() => {window.location.href = base + "/"}}><span class="material-symbols-outlined" translate="no">arrow_circle_left</span></button>
    {:else}
    <h1 style="font-size: 100px;">{errorMessage[0]}</h1>
    <h2 style="margin-bottom: 100px">{errorMessage[1]}</h2>
    
    <h4><button style="font-size: 15px;" onclick={() => {window.location.href = base + "/"}}>Return Home</button></h4>
    <p>You can also use <span style:font-family="Montserrat, Space Grotesk, Futura" style:font-weight = 700>ESC</span> Key</p>
    {/if}
{/if}