<script lang="ts">
    import katex from "katex";
    import { game } from "@/stores/writeGame";
    import { peer } from "@/stores/writePeerObj";
    import { peers } from "@/stores/writePeers";
    import type { ZendoGameMessages } from "@/schemas/messages";

    // You can provide either a number or a string, but not both.
    // Passing a number assumes the koan already exists in the game object.
    export let koanNum: number = undefined;
    export let koanStr: string = undefined;
    let htbn: boolean;
    if (koanStr !== undefined) {
        koanNum = undefined;
        htbn = undefined;
    }
    if (koanNum !== undefined) {
        koanStr = $game.koans[koanNum].string;
        htbn = $game.koans[koanNum].htbn;
    }

    const pushGame = () => {
        const msg: ZendoGameMessages = {
            type: "gameReplace",
            game: JSON.stringify($game)
        }
        for (const p of $peers) {
            p.connection.send(msg);
        }
    };

    const toggleNature = () => {
        $game.koans[koanNum].htbn = !$game.koans[koanNum].htbn;
        $game = $game;
        pushGame();
    };

    let equation: string;
    if ($game.koanType === "math") {
        try {
            equation = katex.renderToString(koanStr, {displayMode: true});
        } catch {
            equation = "Invalid equation";
        }
    }

    let modalDelete = "";
    const deleteKoan = () => {
        // Subtract 1 from each guess index greater than the koan number
        if ($game.hasOwnProperty("guesses")) {
            for (let i = 0; i < $game.guesses.length; i++) {
                if ($game.guesses[i].index > koanNum) {
                    $game.guesses[i].index--;
                    $game = $game;
                }
            }
        }
        // Remove koan from the list
        $game.koans.splice(koanNum, 1);
        $game = $game;
        pushGame();
        modalDelete = "";
    };
</script>

{#if ( (koanStr === undefined) && (koanNum === undefined) ) }
    <p class="is-danger">The developer screwed up. The ViewKoan component needs to receive either a number or a string.</p>
{:else}
<article class="media">
    {#if koanNum !== undefined}
        <div class="media-left">
            <p class="subtitle">{koanNum + 1}</p>
        </div>
    {/if}
        <div class="media-content">
        {#if $game.koanType === "image"}
            <figure class="koan image is-square">
                <img src="{koanStr}" alt="Undescribed koan">
            </figure>
        {:else if $game.koanType === "math"}
            <figure class="koan">
                <p>{@html equation}</p>
            </figure>
        {:else}
            <figure class="koan text-koan content">
                <p>{koanStr}</p>
            </figure>
        {/if}
        </div>
    {#if htbn !== undefined}
        <div class="media-right">
        {#if htbn}
            <span class="icon">
                <i class="fa-solid fa-check" aria-hidden="true"></i>
            </span>
        {:else}
            <span class="icon">
                <i class="fa-solid fa-xmark" aria-hidden="true"></i>
            </span>
        {/if}
        {#if $game.master === $peer.id}
            <button class="button is-normal is-responsive" on:click="{toggleNature}">
                <span class="icon" aria-label="Toggle Buddha nature" title="Toggle Buddha nature">
                    <i class="fa-solid fa-right-left" aria-hidden="true"></i>
                </span>
            </button>
            <button class="button is-normal is-responsive" on:click="{() => modalDelete = "is-active"}">
                <span class="icon" aria-label="Delete koan" title="Delete koan">
                    <i class="fa-solid fa-trash-can" aria-hidden="true"></i>
                </span>
            </button>
        {/if}
        </div>
    {/if}
    </article>
{/if}

<div class="modal {modalDelete}" id="confirmDelete">
    <div class="modal-background"></div>
    <div class="modal-card">
        <header class="modal-card-head">
            <p class="modal-card-title">Delete Koan?</p>
        </header>
        <section class="modal-card-body">
            <p><strong>This action cannot be undone!</strong> Avoid deleting koans if possible. It's better to just change its nature. Deleting koans can confuse players in the moment and when looking back at past guesses. You also can't replace them. Any new koan you add in its place will appear at the end of the list.</p>
        </section>
        <footer class="modal-card-foot">
            <button class="button is-success" on:click="{deleteKoan}">Delete Koan</button>
            <button class="button" on:click="{() => {modalDelete = ""}}">Cancel</button>
        </footer>
    </div>
</div>

<style>
    @import url('https://fonts.googleapis.com/css2?family=Inconsolata&display=swap');
    .fa-check {
        color: green;
    }
    .fa-xmark {
        color: red;
    }
    .text-koan {
        font-family: 'Inconsolata', monospace;
    }
</style>