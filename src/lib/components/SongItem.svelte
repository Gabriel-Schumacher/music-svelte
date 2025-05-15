<script lang="ts">
import type { Song } from '$lib/types/song'; // Assuming you have a type definition for Song
export let song: Song;
</script>

<div class="flex gap-4 items-center p-4 bg-gray-100 rounded-md shadow-md">
    <div>
        <img src={song.cover} alt="{song.title} cover" class="w-20 h-20 rounded-md">        
    </div>
    <div>
        <h2 class="text-2xl font-bold mb-2">{song.title}</h2>
        <p class="text-lg">{song.artist}</p>
        <p class="text-sm text-gray-500">{song.album}</p>        
    </div>
</div>