<script lang="ts">
import type { Song } from '$lib/types/song'; // Assuming you have a type definition for Song
export let song: Song;
export let onPlay: () => void;

let isHovered = false;
</script>

<div class="flex gap-4 items-center p-4 bg-gray-100 rounded-md shadow-md">
    <div 
        class="relative"
        role="button"
        tabindex="0"
        onmouseenter={() => isHovered = true}
        onmouseleave={() => isHovered = false}
    >
        <img src={song.cover} alt="{song.title} cover" class="w-20 h-20 rounded-md">
        {#if isHovered}
        <button 
            onclick={onPlay} 
            class="absolute inset-0 bg-black bg-opacity-50 text-white flex items-center justify-center rounded-md cursor-pointer"
        >
            Play
        </button>
        {/if}
    </div>
    <div>
        <h2 class="text-2xl font-bold mb-2">{song.title}</h2>
        <p class="text-lg">{song.artist}</p>
        <p class="text-sm text-gray-500">{song.album}</p>        
    </div>
</div>

<style>
/* Ensure the play button overlay is styled properly */
button {
    transition: opacity 0.2s ease-in-out;
}
</style>