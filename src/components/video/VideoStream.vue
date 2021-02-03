<template>
    <video autoplay="1" ref="videoRef" class="video" />
    <div>I am video</div>
</template>

<script>
import {toRef, ref, watchEffect, onBeforeUnmount} from 'vue';
export default {
    name: 'VideoStream',
    props:{
        track: String
    },
    setup(props){
        const videoRef = ref(null);
        const track = toRef(props, 'track');
        watchEffect( ()=>{
           const element = videoRef.value;
           track.value.attach(element);
        })

        onBeforeUnmount(()=> {
           const element = videoRef.value;
           track.value.detach(element);
        })
        return {
            videoRef
        }
    }
}
</script>


<style scoped>
.video {
    height: 180px;
    width: 320px;
}
</style>