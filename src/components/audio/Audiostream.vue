<template>
    <audio autoplay="1" ref="audioRef" />
</template>

<script>
import { onUpdated, toRef } from 'vue';
export default {
    name: 'AudioStream',
    props:{
        track: String
    },
    setup(props){
        const audioRef = ref(null);
        const track = toRef(props, 'track');

        onUpdated(()=>{
            window.addEventListener('load', ()=>{
                const element = audioRef.currrent;
                track.attach(element);
                return ()=>{
                    track.detach(element);
                }
            })
        })
    }
    
}
</script>