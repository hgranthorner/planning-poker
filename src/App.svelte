<script lang="ts">
    import {createClient} from '@supabase/supabase-js'
    import {env} from './credentials'

    type Player = {
        name: string
        vote?: number
        lastSeen: Date
    }

    const supabase = createClient(env.apiUrl, env.anon)
    let playerName = ''
    let playerVote
    let players: Player[] = []
    setInterval(() => {
        supabase.from('players')
            .select('*')
            .then(({data, error}) => {
                if (error) throw error
                players = data.map((d: { name: string, last_seen: string, vote?: number }) => ({
                    ...d,
                    lastSeen: new Date(d.last_seen)
                }))
                players.sort((p1, p2) => p1.name > p2.name ? 1 : -1)
                return players
            })
            .then(players => {
                for (let p of players) {
                    if (p.lastSeen.getTime() < Date.now() - 30_000) {
                        console.log(p)
                    }
                }
            })
    }, 1000)
    $: if (playerVote) {
        debugger
        console.log('playerVote', playerVote)
        supabase.from('players')
            .update({vote: playerVote})
            .match({name: playerName})
            .then(() => console.log('Updated vote!'))
    }
</script>
<style>
    .current {
        color: dodgerblue;
    }

    .players > li {
        cursor: pointer;
    }
</style>
<main>
    <input bind:value={playerName}>
    <button on:click={() => supabase.from('players').insert({name: playerName}).then(x => console.log(x))}>Enter
    </button>
    <ul class="players">
        {#each players as player}
            <li on:click={() => {
             playerName = player.name
             playerVote = player.vote
            }}
                class="{playerName === player.name ? 'current' : ''}">{player.name}
                {#if player.vote}: {player.vote}{/if}
            </li>
        {/each}
    </ul>
    <div>
        <label for="vote-input">Your vote: </label>
        <input id="vote-input" type="number" bind:value={playerVote}>
    </div>
</main>
