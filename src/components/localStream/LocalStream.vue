<template>
    <div>
        <div  v-for="(track, index) in localTracks" :key="index">
            <div v-if="track.isVideoTrack()">
                <VideoStream :track="track" />
            </div>
            <div v-else>
                <AudioStream :track="track" />
            </div>
        </div>
    </div>
</template>

<script>
import { toRef } from 'vue';
import AudioStream from '../audio/Audiostream';
import VideoStream from '../video/VideoStream';

export default {
    name: 'LocalStream',
    props: {
        localTracks: String
    },
    components: {
        AudioStream,
        VideoStream
    },
    setup(props){
        toRef(props, 'localTracks');
    }
}
</script>
