<template>
    <div>
        <div  v-for="(track, index) in remoteTracks" :key="index">
            <div v-if="track.isVideoTrack()">
                <VideoStream :track="track" />
            </div>
            <div v-else>
                <AudioStream :track="track" />
            </div>
        </div>
        <div>I am remote</div>
    </div>
</template>

<script>
import { toRef } from 'vue';
import AudioStream from '../audio/Audiostream';
import VideoStream from '../video/VideoStream';

export default {
    name: 'RemoteStream',
    props: {
        remoteTracks: [Function, Array]
    },
    components: {
        AudioStream,
        VideoStream
    },
    setup(props){
       toRef(props, 'remoteTracks');
    }
}
</script>