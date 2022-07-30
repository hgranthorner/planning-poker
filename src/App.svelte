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
</script>
<style></style>
<main>
    <input bind:value={playerName}>
    <button on:click={() => supabase.from('players').insert({name: playerName}).then(x => console.log(x))}>Enter
    </button>
    <ul>
        {#each players as player}
            <li>{player.name}</li>
        {/each}
    </ul>
</main>
