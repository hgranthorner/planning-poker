<script lang="ts">
    import {createClient} from '@supabase/supabase-js'
    import {env} from './credentials'

    const supabase = createClient(env.apiUrl, env.anon)
</script>
<style></style>
{#await supabase.from('players').select('*').then(({data, error}) => {
    if (error) throw error
    return data
})}
    fetching...
{:then data}
    {JSON.stringify(data)}
{/await}