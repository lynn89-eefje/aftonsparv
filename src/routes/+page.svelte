<script>
    import { onMount } from "svelte";
    import { fly, slide } from "svelte/transition";
    import { base } from "$app/paths";

    let intro = $state(false);

    let addSession = $state(true);
    let newName = $state("");
    let slackID = $state("");

    let sessionsLocked = $state(false);
    $effect(() => {
        if (sessionsLocked) {
            addSession = false;
        }
    })

    let sessions = $state([]);

    onMount(function() {
        setTimeout(() => {intro = true;}, 100);
        setTimeout(() => {document.getElementById("newSessionInput").focus()}, 200);
    }) 

    function newSession() {
        event.preventDefault();
        if (sessions.indexOf(newName) == -1) {
            sessions.push(newName);
            newName = "";
            addSession = false;
            setTimeout(() => {addSession = true;}, 500);
            setTimeout(() => {document.getElementById("newSessionInput").focus(); scrollTo(document.getElementById("newSessionInput").y)}, 600)
        }
    }

    function removeSession(target) {
        let newList = []
        for (let i = 0; i < sessions.length; i++) {
            if (sessions[i] != target) {
                newList.push(sessions[i]);
            }
        }
        sessions = newList;
    }

    function next() {
        scrollTo(document.getElementById("idInput").y);
        let sessionsString = "";
        if (sessions.length > 1) {
            for (let i = 0; i < sessions.length; i++) {
                sessionsString += sessions[i] + "[x]";
            }
        }
        else if (sessions.length == 1) {
            sessionsString = sessions[0];
        }

        window.location.href = base + "/" + slackID + "/" + sessionsString;
    }
    onMount(function() {
        window.addEventListener("keydown", function(event) {
            if (event.key == "Escape") {
                sessionsLocked = true;
            }
            this.setTimeout(() => {this.document.getElementById("idInput").focus(); this.scrollTo(this.document.getElementById("idInput")).y;}, 100)
        })
    })
</script>
<svelte:head>   
    <title>Aftonsparv</title>
</svelte:head>
<style>
    .field {
        background-color: rgb(82, 95, 82);
        margin: 20px;
        border-radius: 30px;
        padding: 20px;
        position: relative;
    }
    .session {
        background-color: rgb(127, 134, 95);
        padding: 10px;
        border-radius: 30px;
        margin: 20px;
        border: solid 10px rgb(191, 199, 160);
    }

    div.session button.cancelButton {
        transform: scale(0.8), translateY(5px) !important;
    }
    div.session button.cancelButton:hover {
        box-shadow: 0px 0px 5px 2px rgba(64, 77, 71, 0.438);
    }
    div.session button.cancelButton:active {
        transform: scale(0.7);
    }
</style>
{#if intro}
    <h1 id="title" transition:slide>AFTONSPARV</h1>
    <h3 style="font-weight: 200; margin-bottom: 80px;">Hackatime Viewer</h3>

    <div class="field" transition:fly={{y:200, delay: 500}}>
        <h2>Hackatime Sessions</h2>
        {#each sessions as x}
            <div transition:slide class="session"><h3>{x} {#if !sessionsLocked}<button class="cancelButton" onclick={() => {removeSession(x)}}><span class="material-symbols-outlined" translate="no">cancel</span></button>{/if}</h3></div>
        {/each}
        {#if addSession}
        <form transition:slide onsubmit={newSession}>
            <h3>Enter Session Name</h3>
            <input id="newSessionInput" required type="text" placeholder="Type here" bind:value={newName}/>
            <input type="submit" value="Add">
            <button class="cancelButton" onclick={() => {addSession = false;}} title="Cancel"><span class="material-symbols-outlined" translate="no">cancel</span></button>
            <p>*Session name is case sensitive</p>
        </form>
        {/if}
        {#if sessions.length == 0 && !addSession}
        <h3><i>None selected</i></h3>
        {/if}
        {#if !sessionsLocked}<h3 id="buttons"><button onclick={() => {addSession = true; setTimeout(() => {document.getElementById("newSessionInput").focus()}, 100);}}><span title="Add Session" translate = "no" class="material-symbols-outlined">add_circle</span></button><button onclick={() => {sessionsLocked = true; setTimeout(() => {document.getElementById("idInput").focus(); scrollTo(document.getElementById("idInput").y)}, 100);}}><span title="Confirm Sessions" translate = "no" class="material-symbols-outlined">check_circle</span></button></h3>
        <p>If you wish to see overall user statistics, confirm without selecting any sessions. Use the <span style:font-family="Montserrat, Space Grotesk, Futura" style:font-weight = 700>ESC</span> keybind to confirm, or use the button.</p>{/if}
        {#if sessionsLocked}<h4 transition:slide={{axis:"x"}} style="position: absolute; top: 5px; right: 20px"><span style="font-weight: 700" translate="no" class="material-symbols-outlined">lock</span></h4>{/if}
    </div>

    {#if sessionsLocked}
        <div class="field" transition:fly={{y:200}}>
            <h2>Slack ID</h2>
            <form onsubmit={next}>
                <h3>Enter Slack ID</h3>
                <input required type="text" placeholder="Type here" id="idInput" bind:value={slackID}/>
                <input type="submit" value="Enter ID" />
                <p>If you're not sure what the ID of an user is, use the <i>#what-is-my-slack-id</i> channel!</p>
            </form>
        </div>
    {/if}

{/if}

