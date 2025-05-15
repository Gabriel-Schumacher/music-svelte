<script lang="ts">
    import Player from '$lib/components/Player.svelte';
    import SongItem from '$lib/components/SongItem.svelte';
    import { onMount } from 'svelte';
    import type { Song } from '$lib/types/song'; // Assuming you have a type definition for Song



    // Use const for immutable data
    const library: Song[] = [
        {
            title: 'Bleecker Street',
            artist: 'Simon & Garfunkel',
            album: 'Wednesday Morning, 3 A.M.',
            cover: '/src/lib/Media/BleeckerSteet.jpg',
            url: '/src/lib/Media/BleeckerStreet.mp3'
        },
        {
            title: 'Bright Lights',
            artist: 'The Killers',
            album: 'Single',
            cover: '/src/lib/Media/BrightLights.jpeg',
            url: '/src/lib/Media/BrightLights.mp3'
        },
        {
            title: 'Hound Dog',
            artist: 'Elvis Presley',
            album: 'Elvis (1956)',
            cover: '/src/lib/Media/HoundDog.png',
            url: '/src/lib/Media/HoundDog.mp3'
        },
    ];

    // For static assets, in SvelteKit 5 it's better to use the $lib alias
    // If you move the media files to static directory, use paths like:
    // url: '/media/HoundDog.mp3'
</script>

<div class="container mx-auto p-4">
    <h1 class="text-3xl font-bold mb-8 text-center">Music Player</h1>
    <Player songs={library} />
</div>
<div class="container mx-auto p-4">
    <ul class="flex flex-col gap-4">
        {#each library as song}
            <li>
                <SongItem song={song} />
            </li>
        {/each}
    </ul>
</div>

