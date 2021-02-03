<template>
    <audio autoplay="1" ref="audioRef" />
</template>

<script>
import {toRef, ref, watchEffect, onBeforeUnmount} from 'vue';
export default {
    name: 'AudioStream',
    props:{
        track: String
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