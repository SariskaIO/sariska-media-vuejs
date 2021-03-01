<template>
    <audio autoplay="1" ref="audioRef" :src="track"></audio>
</template>

<script>
import {toRef, ref, watchEffect, onBeforeUnmount} from 'vue';
export default {
    name: 'AudioStream',
    props:{
        track: {
          type: [String, Array, Object]
        } 
    },
    setup(props){
        const audioRef = ref(null);
        const track = toRef(props, 'track');

        watchEffect( ()=>{
          const element = audioRef.value;
          track.value.attach(element);
        })

        onBeforeUnmount(()=> {
          const element = audioRef.value;
          track.value.detach(element);

        })
        return {
          audioRef
        }
    }
}
</script>