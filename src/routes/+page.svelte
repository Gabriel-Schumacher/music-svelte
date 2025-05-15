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
            album: 'The Tortured Poets Society',
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
        currentSong = library[currentIndex]; // Ensure currentSong is updated
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

        currentIndex = (currentIndex + 1) % library.length;
        currentSong = library[currentIndex]; // Update currentSong before setup
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
        console.log('Next track:', currentSong.title);
    }

    function playPrevious() {
        if (isLoading || library.length <= 1) return;

        currentIndex = (currentIndex - 1 + library.length) % library.length;
        currentSong = library[currentIndex]; // Update currentSong before setup
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
        console.log('Previous track:', currentSong.title);
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
        currentTime = time; // Update currentTime to reflect the seek
        console.log('Seeked to:', time);
    }

    function playSong(song: Song) {
        const index = library.findIndex((s) => s.url === song.url);
        if (index === -1 || isLoading) return;

        currentIndex = index;
        currentSong = library[currentIndex]; // Update currentSong before setup
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
        console.log('Playing selected track:', currentSong.title);
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

