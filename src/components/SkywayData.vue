<template>
  <div class="container">
      <h1 class="heading">P2P Data example</h1>
      <p class="note">
        Enter remote peer ID to connect.
      </p>
      <div class="p2p-data">
        <div>
          <p>Your ID: {{localId}}</p>
          <input type="text" placeholder="Remote Peer ID" v-model="remoteId">
          <button ref="connectTrigger">Connect</button>
          <button ref="closeTrigger">Close</button>
        </div>
        <div>
          <pre class="messages">
            <span v-for="msg, i in messages" :key="i">{{msg}}<br></span>
          </pre>
          <input type="text" v-model="localText">
          <button ref="sendTrigger">Send</button>
        </div>
      </div>
    </div>
</template>

<script lang="ts">
import { defineComponent, ref, reactive, onMounted } from 'vue'
import Peer from 'skyway-js'

export default defineComponent({
    name: 'SkywayData',
    setup() {

      const localId = ref('')
      const localText = ref('')
      const connectTrigger = ref<HTMLButtonElement>()
      const closeTrigger = ref<HTMLButtonElement>()
      const sendTrigger = ref<HTMLButtonElement>()
      const remoteId = ref('')
      const messages = reactive<string[]>([])
      const peer = ref<Peer | undefined>()


      onMounted(async () => {
        if (!connectTrigger.value) return

        peer.value = new Peer({
          key: 'e01b04e5-6d3f-412d-9eb4-e290ab52b643',
          debug: 3,
        })

        connectTrigger.value.addEventListener('click', () => {
          if (!peer.value || !peer.value.open) return
          if (!closeTrigger.value) return

          const dataConnection = peer.value.connect(remoteId.value)

          const onClickSend = () => {
            if (!dataConnection) return
            const data = localText.value
            dataConnection.send(data)
            messages.push(`You: ${data}`)
            localText.value = ''
          }

          dataConnection.once('open', async () => {
            if (!sendTrigger.value) return
            messages.push(`=== DataConnection has been opened ===`)
            sendTrigger.value.addEventListener('click', onClickSend)
          })

          dataConnection.on('data', data => {
            messages.push(`Remote: ${data}`)
          })

          dataConnection.once('close', () => {
            if (!sendTrigger.value) return
            messages.push(`=== DataConnection has been closed ===`)
            sendTrigger.value.removeEventListener('click', onClickSend)
          })

          closeTrigger.value.addEventListener('click', () => dataConnection.close(true), {
            once: true,
          })

        })

        peer.value.once('open', id => (localId.value = id))
        
        peer.value.on('connection', dataConnection => {
          if (!closeTrigger.value) return

          const onClickSend = () => {
            const data = localText.value
            dataConnection.send(data)
            messages.push(`You: ${data}`)
            localText.value = ''
          }

          dataConnection.once('open', async () => {
            if (!sendTrigger.value) return
            messages.push(`=== DataConnection has been opened ===`)
            sendTrigger.value.addEventListener('click', onClickSend)
          })

          dataConnection.on('data', data => {
            messages.push(`Remote: ${data}`)
          })

          dataConnection.once('close', () => {
            if (!sendTrigger.value) return
            messages.push(`=== DataConnection has been closed ===`)
            sendTrigger.value.removeEventListener('click', onClickSend)
          })

          closeTrigger.value.addEventListener('click', () => dataConnection.close(true), {
            once: true,
          })

        })

        peer.value.on('error', console.error)
        
      })
      
      return {
        localId,
        localText,
        connectTrigger,
        closeTrigger,
        sendTrigger,
        remoteId,
        messages,
        peer,
      }
    }
})
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