<template>
    <div class="container">
        <h1 class="heading">P2P Media example</h1>
        <p class="note">
            Enter remote peer ID to call.
        </p>
        <div class="p2p-media">
            <div class="remote-stream">
                <video ref="remoteVideo" playsinline></video>
            </div>
            <div class="local-stream">
                <video ref="localVideo" playsinline></video>
                <p>Your ID: {{ localId }}</p>
                <input type="text" placeholder="Remote Peer ID" v-model="remoteId">
                <button ref="callTrigger">Call</button>
                <button ref="closeTrigger">Close</button>
            </div>
        </div>
        <p class="meta" id="js-meta"></p>
    </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from 'vue'
import Peer from 'skyway-js'

export default defineComponent({
    name: 'SkywayVideo',
    setup() {

        const localVideo = ref<HTMLMediaElement>()
        const remoteVideo = ref<HTMLMediaElement>()
        const callTrigger = ref<HTMLButtonElement>()
        const closeTrigger = ref<HTMLButtonElement>()
        const peer = ref<Peer | undefined>()
        const remoteId = ref('')
        const localId = ref('')

        onMounted(async () => {
            
            const localStream = await navigator.mediaDevices.getUserMedia({ audio: true, video: true, })

            if (!localVideo.value) return
            localVideo.value.muted = true
            localVideo.value.srcObject = localStream
            await localVideo.value.play()

            peer.value = new Peer({
                key: 'e01b04e5-6d3f-412d-9eb4-e290ab52b643',
                debug: 3,
            })

            if (!callTrigger.value) return
            callTrigger.value.addEventListener('click', () => {
                if (!peer.value || !peer.value.open) return
                const mediaConnection = peer.value.call(remoteId.value, localStream)

                mediaConnection.on('stream', async stream => {
                    if (!remoteVideo.value) return
                    remoteVideo.value.srcObject = stream
                    await remoteVideo.value.play()
                })

                mediaConnection.once('close', () => {
                    if (!remoteVideo.value) return
                    remoteVideo.value.srcObject = null
                })

                if (!closeTrigger.value) return
                closeTrigger.value.addEventListener('click', () => mediaConnection.close(true))

            })

            peer.value.once('open', id => (localId.value = id))

            peer.value.on('call', mediaConnection => {
                mediaConnection.answer(localStream)

                mediaConnection.on('stream', async stream => {
                    if (!remoteVideo.value) return
                    remoteVideo.value.srcObject = stream
                    await remoteVideo.value.play()
                })

                mediaConnection.once('close', () => {
                    if (!remoteVideo.value) return
                    remoteVideo.value.srcObject = null
                })
                
                if (!closeTrigger.value) return
                closeTrigger.value.addEventListener('click', () => mediaConnection.close(true))
            })

            peer.value.on('error', console.error)
        })

        return {
            localVideo,
            remoteVideo,
            callTrigger,
            closeTrigger,
            peer,
            remoteId,
            localId,
        }
    }
});
</script>

<style scoped>
/* global styles */
video {
  background-color: #111;
  width: 100%;
}

.heading {
  text-align: center;
  margin-bottom: 0;
}

.note {
  text-align: center;
}

.meta {
  text-align: center;
  font-size: .8rem;
  color: gray;
}

.container {
  margin-left: auto;
  margin-right: auto;
  width: 980px;
}

/* p2p-media styles */
.p2p-media {
  display: flex;
  align-items: center;
  flex-direction: column;
}

.p2p-media .remote-stream {
  width: 50%;
}

.p2p-media .local-stream {
  width: 30%;
}

/* p2p-data styles */
.p2p-data {
  display: grid;
  grid-template-columns: 30% 1fr;
  margin: 0 8px;
}

.p2p-data .messages {
  background-color: #eee;
  min-height: 100px;
  padding: 8px;
  margin-top: 0;
}

/* room */
.room {
  display: grid;
  grid-template-columns: 30% 40% 30%;
  gap: 8px;
  margin: 0 8px;
}

.room .remote-streams {
  background-color: #f6fbff;
}

.room .messages {
  background-color: #eee;
  min-height: 100px;
  padding: 8px;
  margin-top: 0;
}
</style>