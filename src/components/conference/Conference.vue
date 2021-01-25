<template>
    <div>
        <LocalStream :room="room" />
        <RemoteStream :room="room" />
    </div>
</template>

<script>
import { onBeforeUnmount, toRef, watchEffect, ref } from 'vue';
import JitsiMeetJS from "sariska-media-transport";
import {conferenceConfig} from '@/constants';
import LocalStream from '../localStream/LocalStream';
import RemoteStream from '../remoteStream/RemoteStream';

export default {
    name: 'Conference',
    components: {
        LocalStream,
        RemoteStream
    },

    props: {
        connection: String
    },

    setup(props){

        const room = ref(null);
        const remoteTracks = ref([]);
        const localTracks = ref([]);
        const connection = toRef(props, 'connection');

        watchEffect(()=>{
            if (room.value && localTracks.value.length) {
               localTracks.value.forEach(track=>room.value.addTrack(track).catch(err =>console.log("track already added", err)));
            }
            JitsiMeetJS.createLocalTracks({devices:["audio", "video"], resolution: "180"}).
            then(tracks => {
                localTracks.value = tracks;
            }).
            catch(()=>console.log("failed to fetch tracks"));
        })

        watchEffect(()=>{
            if ( connection.value === null ) {
               return;
            }
            const room = connection.value.initJitsiConference(conferenceConfig);
            const onConferenceJoined = ()=> {
               room.value = room;
            }

            const onTrackRemoved = (track)=> {
               remoteTracks.value = remoteTracks.value.filter(item => item.track.id !== track.track.id);
            }

            const onRemoteTrack = (track)=> {
               if (!track  || track.isLocal()) {
                  return;
               }
               remoteTracks.value = remoteTracks.value.push(track);
            }

            room.on(JitsiMeetJS.events.conference.CONFERENCE_JOINED, onConferenceJoined);
            room.on(JitsiMeetJS.events.conference.TRACK_ADDED, onRemoteTrack);
            room.on(JitsiMeetJS.events.conference.TRACK_REMOVED, onTrackRemoved);
            console.log('joineddmmddmdmdmdmd')
            room.join();
          })

          onBeforeUnmount(()=>{
             if (room.value && room.value.isJoined()) {
                 room.value.leave().then(() => connection.value.disconnect(event));
             }
          })

        return {
            room
        }
    }
}
</script>