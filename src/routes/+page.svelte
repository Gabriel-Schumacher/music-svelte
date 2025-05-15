<script lang="ts">
    import Player from '$lib/components/Player.svelte';
    import SongItem from '$lib/components/SongItem.svelte';
    import { onMount } from 'svelte';
    import type { Song } from '$lib/types/song';

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

    let currentIndex = 0;
    let audio: HTMLAudioElement;
    let isPlaying = false;
    let isLoading = false;
    let currentTime = 0;
    let duration = 0;

    $: currentSong = library[currentIndex];

    onMount(() => {
        audio = new Audio();
        audio.preload = 'auto';

        audio.addEventListener('timeupdate', handleTimeUpdate);
        audio.addEventListener('loadedmetadata', handleLoadedMetadata);
        audio.addEventListener('ended', playNext);
        audio.addEventListener('error', () => {
            console.error('Audio error occurred');
            isLoading = false;
        });

        setupCurrentSong();
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

        if (audio.paused) {
            audio.play().then(() => {
                isPlaying = true;
            }).catch(error => {
                console.error('Play error:', error);
            });
        } else {
            audio.pause();
            isPlaying = false;
        }
    }

    function playNext() {
        if (isLoading || library.length <= 1) return;

        currentIndex = (currentIndex + 1) % library.length;
        setupCurrentSong();

        audio.oncanplay = () => {
            audio.oncanplay = null;
            isLoading = false;

            if (isPlaying) {
                audio.play().catch(error => {
                    console.error('Failed to play next track:', error);
                });
            }
        };
    }

    function playPrevious() {
        if (isLoading || library.length <= 1) return;

        currentIndex = (currentIndex - 1 + library.length) % library.length;
        setupCurrentSong();

        audio.oncanplay = () => {
            audio.oncanplay = null;
            isLoading = false;

            if (isPlaying) {
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
</script>

<div class="container mx-auto p-4">
    <h1 class="text-3xl font-bold mb-8 text-center">Music Player</h1>
    <Player
        songs={library}
        currentSong={currentSong}
        isPlaying={isPlaying}
        isLoading={isLoading}
        currentTime={currentTime}
        duration={duration}
        onTogglePlay={togglePlay}
        onPlayNext={playNext}
        onPlayPrevious={playPrevious}
    />
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

