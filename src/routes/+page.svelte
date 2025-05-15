<script lang="ts">
    import { writable } from 'svelte/store';
    import Player from '$lib/components/Player.svelte';
    import SongItem from '$lib/components/SongItem.svelte';
    import { onMount } from 'svelte';
    import type { Song } from '$lib/types/song';

    const library: Song[] = [
        {
            title: 'Bleecker Street',
            artist: 'Simon & Garfunkel',
            album: 'Wednesday Morning, 3 A.M.',
            cover: '/BleeckerSteet.jpg',
            url: '/BleeckerStreet.mp3',
            genre: 'Folk Rock',
        },
        {
            title: 'Bright Lights',
            artist: 'The Killers',
            album: 'Single',
            cover: '/BrightLights.jpeg',
            url: '/BrightLights.mp3',
            genre: 'Alternative Rock',
        },
        {
            title: 'Hound Dog',
            artist: 'Elvis Presley',
            album: 'Elvis (1956)',
            cover: '/HoundDog.png',
            url: '/HoundDog.mp3',
            genre: 'Rock and Roll',
        },
        {
            title: 'Fortnite',
            artist: 'Taylor Swift',
            album: 'The Tortured Poet\'s Society',
            cover: '/Fortnite.webp',
            url: '/Fortnite.mp3',
            genre: 'Pop',
        },
        {
            title: 'I Had Some Help',
            artist: 'Morgan Wallen & Post Malone',
            album: 'F-1 Trillion',
            cover: '/HadHelp.webp',
            url: '/HadHelp.mp3',
            genre: 'Country',
        },
    ];

    let playlists = [];

    // Create a writable store for currentIndex with persistence in localStorage
    function createPersistentIndex() {
        const savedIndex = typeof window !== 'undefined' ? localStorage.getItem('currentIndex') : null;
        const initialIndex = savedIndex !== null ? parseInt(savedIndex, 10) : 0;

        const { subscribe, set, update } = writable(initialIndex);

        subscribe((value) => {
            if (typeof window !== 'undefined') {
                localStorage.setItem('currentIndex', value.toString());
            }
        });

        return { subscribe, set, update };
    }

    export const currentIndex = createPersistentIndex();

    let audio: HTMLAudioElement;
    let isPlaying = false;
    let isLoading = false;
    let currentTime = 0;
    let duration = 0;

    $: currentSong = library[$currentIndex]; // Derive currentSong dynamically

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
        if (!audio || !library[$currentIndex]) return;

        isLoading = true;
        audio.pause();
        audio.currentTime = 0;
        audio.src = library[$currentIndex].url;
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
        console.log('Play/Pause toggled:', isPlaying, currentSong.title);
    }

    function playNext() {
        if (isLoading || library.length <= 1) return;

        currentIndex.update((index) => (index + 1) % library.length);
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
        console.log('Next track:', library[$currentIndex].title);
    }

    function playPrevious() {
        if (isLoading || library.length <= 1) return;

        currentIndex.update((index) => (index - 1 + library.length) % library.length);
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
        console.log('Previous track:', library[$currentIndex].title);
    }

    function handleTimeUpdate() {
        currentTime = audio.currentTime;
    }

    function handleLoadedMetadata() {
        duration = audio.duration;
        isLoading = false;
    }

    function seekTo(time: number) {
        if (!audio || isLoading) return;

        audio.currentTime = time;
        currentTime = time;
        console.log('Seeked to:', time);
    }

    function playSong(song: Song) {
        const index = library.findIndex((s) => s.url === song.url);
        if (index === -1 || isLoading) return;

        currentIndex.set(index);
        setupCurrentSong();

        audio.oncanplay = () => {
            audio.oncanplay = null;
            isLoading = false;

            audio.play().then(() => {
                isPlaying = true;
            }).catch(error => {
                console.error('Failed to play selected track:', error);
            });
        };
        console.log('Playing selected track:', library[$currentIndex].title);
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
        onSeekTo={seekTo} 
    />
</div>
<div class="container mx-auto p-4">
    <ul class="flex flex-col gap-4">
        {#each library.slice().sort((a, b) => a.title.localeCompare(b.title)) as song}
            <li>
                <SongItem song={song} onPlay={() => playSong(song)} />
            </li>
        {/each}
    </ul>
</div>

