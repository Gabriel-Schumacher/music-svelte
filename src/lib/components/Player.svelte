<script lang="ts">
    import { onMount, onDestroy } from 'svelte';
    import type { Song } from '$lib/types/song';

    export let songs: Song[] = [];

    let currentIndex = 0;
    let audio: HTMLAudioElement;
    let currentTime = 0;
    let duration = 0;
    let isLoading = false;
    let audioContext: AudioContext | null = null;
    let audioSource: MediaElementAudioSourceNode | null = null;

    // Use the $ syntax for derived values
    $: currentSong = songs[currentIndex];
    $: isPlaying = audio && !audio.paused; // Automatically derive isPlaying from audio state

    onMount(() => {
        audio = new Audio();
        audio.preload = 'auto';

        try {
            audioContext = new (window.AudioContext || (window as any).webkitAudioContext)();
            if (audioContext) {
                audioSource = audioContext.createMediaElementSource(audio);
                audioSource.connect(audioContext.destination);
            }
        } catch (e) {
            console.error('AudioContext not supported', e);
        }

        audio.addEventListener('timeupdate', handleTimeUpdate);
        audio.addEventListener('loadedmetadata', handleLoadedMetadata);
        audio.addEventListener('ended', handleEnded);
        audio.addEventListener('error', () => {
            console.error('Audio error occurred');
            isLoading = false;
        });

        if (songs.length > 0) {
            setupCurrentSong();
        }
    });

    onDestroy(() => {
        if (audio) {
            audio.removeEventListener('timeupdate', handleTimeUpdate);
            audio.removeEventListener('loadedmetadata', handleLoadedMetadata);
            audio.removeEventListener('ended', handleEnded);
            audio.pause();
            audio.src = '';
        }

        if (audioContext) {
            audioContext.close();
        }
    });

    function setupCurrentSong() {
        if (!audio || !currentSong) return;

        isLoading = true;
        audio.pause();
        audio.currentTime = 0;
        audio.src = currentSong.url;
        audio.load();
    }

    function togglePlay() {
        if (!audio || isLoading) return;

        if (audioContext && audioContext.state === 'suspended') {
            audioContext.resume();
        }

        if (audio.paused) {
            audio.play().catch(error => {
                console.error('Play error:', error);
            });
        } else {
            audio.pause();
        }
    }

    function playNext() {
        if (isLoading || songs.length <= 1) return;

        currentIndex = (currentIndex + 1) % songs.length;
        setupCurrentSong();

        audio.oncanplay = () => {
            audio.oncanplay = null;
            isLoading = false;

            if (!audio.paused) {
                audio.play().catch(error => {
                    console.error('Failed to play next track:', error);
                });
            }
        };
    }

    function playPrevious() {
        if (isLoading || songs.length <= 1) return;

        currentIndex = (currentIndex - 1 + songs.length) % songs.length;
        setupCurrentSong();

        audio.oncanplay = () => {
            audio.oncanplay = null;
            isLoading = false;

            if (!audio.paused) {
                audio.play().catch(error => {
                    console.error('Failed to play previous track:', error);
                });
            }
        };
    }

    function handleTimeUpdate() {
        currentTime = audio.currentTime;
    }

    function handleLoadedMetadata() {
        duration = audio.duration;
        isLoading = false;
    }

    function handleEnded() {
        playNext();
    }

    function seekTo(e: Event) {
        const target = e.target as HTMLInputElement;
        const seekTime = Number(target.value);
        if (audio) {
            audio.currentTime = seekTime;
        }
    }

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
                    oninput={seekTo}
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
                    onclick={playPrevious}
                    aria-label="Previous track"
                    disabled={isLoading}
                >
                    ⏮️
                </button>
                <button 
                    class="p-2 rounded-full bg-gray-700 hover:bg-gray-600 disabled:opacity-50 disabled:cursor-not-allowed" 
                    onclick={togglePlay}
                    aria-label={isPlaying ? 'Pause' : 'Play'}
                    disabled={isLoading}
                >
                    {isPlaying ? '⏸️' : '▶️'}
                </button>
                <button 
                    class="p-2 rounded-full bg-gray-700 hover:bg-gray-600 disabled:opacity-50 disabled:cursor-not-allowed" 
                    onclick={playNext}
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