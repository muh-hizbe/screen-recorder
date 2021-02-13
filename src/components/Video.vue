<template>
    <video id="player" playsinline controls data-plyr-config='{ "hideControls": false, "resetOnEnd": true }'
        class="border-2 border-black rounded">
        <source :src="src" type="video/mp4" />
        <!-- <source :src="src" type="video/webm" /> -->

        <!-- Captions are optional -->
        <track kind="captions" label="English captions" src="/path/to/captions.vtt" srclang="id" default />
    </video>
    <!-- <video autoplay controls :src="src" /> -->
</template>

<script>
import Plyr from 'plyr';
const player = new Plyr('#player');
export default {
    props: ['src']
}
</script>