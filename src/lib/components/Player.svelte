<script lang="ts">
    import type { Song } from '$lib/types/song';

    export let songs: Song[] = [];
    export let currentSong: Song;
    export let isPlaying: boolean;
    export let isLoading: boolean;
    export let currentTime: number;
    export let duration: number;
    export let onTogglePlay: () => void;
    export let onPlayNext: () => void;
    export let onPlayPrevious: () => void;

    function formatTime(seconds: number): string {
        const mins = Math.floor(seconds / 60);
        const secs = Math.floor(seconds % 60);
        return `${mins}:${secs.toString().padStart(2, '0')}`;
    }
</script>

{#if songs.length > 0}
    <div class="player-container p-4 rounded-lg shadow-lg bg-gray-800 text-white">
        <div class="flex flex-col items-center mb-4">
            <div class="relative w-40 h-40 mb-4">
                <img src={currentSong.cover} alt="{currentSong.title} cover" class="w-full h-full rounded-md">
                {#if isLoading}
                    <div class="absolute inset-0 bg-black bg-opacity-50 flex items-center justify-center rounded-md">
                        <div class="animate-spin h-8 w-8 border-4 border-white border-t-transparent rounded-full"></div>
                    </div>
                {/if}
            </div>
            
            <div class="text-center mb-2">
                <h2 class="text-xl font-bold">{currentSong.title}</h2>
                <p class="text-gray-300">{currentSong.artist}</p>
                <p class="text-gray-400 text-sm">{currentSong.album}</p>
            </div>
            
            <div class="w-full mb-4">
                <input 
                    type="range" 
                    min="0" 
                    max={duration || 0} 
                    value={currentTime} 
                    oninput={(e) => onSeek(e)}
                    class="w-full cursor-pointer"
                >
                <div class="flex justify-between text-xs text-gray-400">
                    <span>{formatTime(currentTime)}</span>
                    <span>{formatTime(duration)}</span>
                </div>
            </div>
            
            <div class="flex justify-center space-x-4">
                <button 
                    class="p-2 rounded-full bg-gray-700 hover:bg-gray-600 disabled:opacity-50 disabled:cursor-not-allowed" 
                    onclick={onPlayPrevious}
                    aria-label="Previous track"
                    disabled={isLoading}
                >
                    ⏮️
                </button>
                <button 
                    class="p-2 rounded-full bg-gray-700 hover:bg-gray-600 disabled:opacity-50 disabled:cursor-not-allowed" 
                    onclick={onTogglePlay}
                    aria-label={isPlaying ? 'Pause' : 'Play'}
                    disabled={isLoading}
                >
                    {isPlaying ? '⏸️' : '▶️'}
                </button>
                <button 
                    class="p-2 rounded-full bg-gray-700 hover:bg-gray-600 disabled:opacity-50 disabled:cursor-not-allowed" 
                    onclick={onPlayNext}
                    aria-label="Next track"
                    disabled={isLoading}
                >
                    ⏭️
                </button>
            </div>
        </div>
    </div>
{:else}
    <div class="text-center text-gray-500">
        No songs available in the library.
    </div>
{/if}

<style>
    button[disabled] {
        opacity: 0.5;
        cursor: not-allowed;
    }
</style>