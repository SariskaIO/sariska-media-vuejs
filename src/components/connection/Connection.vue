<template>
    <div>
        <Conference :connection="connection" />
    </div>
</template>

<script>
import Conference from "../conference/Conference.vue";
import SariskaMediaTransport from 'sariska-media-transport';
import {connectionConfig, initSDKConfig} from '../../constants';
import {getToken} from '../../utils/index';
import { onMounted, ref, onBeforeUnmount } from 'vue';

export default {
    name: 'Connection',
    components: {
        Conference
    },
    setup(){
        const connection = ref(null);

                const updateNetwork = ()=>{  //  set internet connectivity status
                    SariskaMediaTransport.setNetworkInfo({isOnline: window.navigator.onLine});
                }



        onMounted(()=>{
                SariskaMediaTransport.init(initSDKConfig);
                SariskaMediaTransport.setLogLevel(SariskaMediaTransport.logLevels.ERROR); //TRACE ,DEBUG, INFO, LOG, WARN, ERROR
                let conn;

                const fetchData =  async ()=>{
                    const token = await getToken();
                    if (!token) {
                        return;
                    }
                    conn = new SariskaMediaTransport.JitsiConnection(token, connectionConfig);
                    conn.addEventListener(SariskaMediaTransport.events.connection.CONNECTION_ESTABLISHED, onConnectionSuccess);
                    conn.addEventListener(SariskaMediaTransport.events.connection.CONNECTION_FAILED, onConnectionFailed);
                    conn.addEventListener(SariskaMediaTransport.events.connection.CONNECTION_DISCONNECTED, onConnectionDisconnected);
                    conn.addEventListener(SariskaMediaTransport.events.connection.PASSWORD_REQUIRED, onConnectionDisconnected);
                    conn.connect();
                }

                const onConnectionSuccess = ()=>{
                    connection.value= conn;
                }

                const onConnectionDisconnected = (error)=>{
                    console.log('connection disconnect!!!', error);
                    if (!connection.value) {
                        return;
                    }
                    connection.value.removeEventListener(
                        SariskaMediaTransport.events.connection.CONNECTION_ESTABLISHED,
                        onConnectionSuccess);
                    connection.value.removeEventListener(
                        SariskaMediaTransport.events.connection.CONNECTION_FAILED,
                        onConnectionFailed);
                    connection.value.removeEventListener(
                        SariskaMediaTransport.events.connection.CONNECTION_DISCONNECTED,
                        onConnectionDisconnected);

                }

                const onConnectionFailed = async (error)=> {
                    console.log('connection failed!!!', error);
                    if (error === "connection.passwordRequired") {  // token expired,  fetch new token and set again
                        const token = await getToken();
                        conn.setToken(token);
                    }
                }

                window.addEventListener("offline", updateNetwork);
                window.addEventListener("online", updateNetwork);

                fetchData();
        })
        onBeforeUnmount(()=>{
                
           window.removeEventListener("offline", updateNetwork);
           window.removeEventListener("online", updateNetwork);
        })

      return {
          connection
      }
    }
}

</script>