<template>
  <div id="app">
    <p>{{connected}}</p>

    {{game.State}}

    <ul>
      <li v-for="player in players" :key="player.Id">
        {{player}}
      </li>
    </ul>

    <button v-if="game.State === 'lobby'" @click="ready()">ready</button>

    <div v-if="game.State === 'guessing'">
      <input type="text" placeholder="Clue" v-model="clue">
      <button @click="submit(clue)">submit</button>
    </div>

    <div v-if="game.State === 'lobby'">
      <input type="text" v-model="newName" placeholder="Name">
      <button @click="rename(newName)">save</button>
    </div>

    <p>Game ID: {{game.Id}}</p>
    <button @click="join('')">start a new game</button>
    <label for="join">Join a game</label>
    <div>
      <input type="number" v-model="input" placeholder="Game ID" id="join">
      <button @click="join(input)">join</button>
    </div>
  </div>
</template>

<script>
  import Player from '@/components/Player'

  export default {
    name: 'app',
    components: {
      Player
    },
    data() {
      return {
        ws: null,
        connected: 0,
        input: '',
        newName: '',
        clue: '',

        game: {
          Id: '',
          Version: -1,
          State: "Loading",
          Win: false
        },

        players: []
      }
    },
    created() {
      let url;
      if (location.protocol === 'https:') {
        url = 'wss://';
      } else {
        url = 'ws://';
      }
      url += location.host + location.pathname + (location.pathname.endsWith('/') ? 'ws' : '/ws');
      this.ws = new WebSocket(url);
      this.ws.onopen = this.onopen.bind(this);
      this.ws.onerror = this.onerror.bind(this);
      this.ws.onclose = this.onclose.bind(this);
      this.ws.onmessage = this.onmessage.bind(this);
    },
    methods: {
      onopen(e) {
        this.connected = 1;
        console.log('here', this.$route.params.id)
        if (this.$route.params.id) {
          this.join(this.$route.params.id);
        } else {
          this.join('');
        }
      },
      onerror(e) {
        console.error(e);
        this.connected = 2;
      },
      onclose(e) {
        this.connected = 2;
      },
      onmessage(e) {
        let data = JSON.parse(e.data);
        switch (data.Type) {
          case "cookie":
            document.cookie = data.Cookie
            console.log("Set cookie", data.Cookie)
            break
          case 'all':
            Object.assign(this.game, data.Update)
            this.players = data.Update.Players
            if (this.$route.params.id !== this.game.Id) {
              this.$router.push({path: this.game.Id})
            }
            break
          default:
            console.error('unknown type', data)
        }
      },
      join(id) {
        this.ws.send(JSON.stringify({Type: "join", Data: id}));
      },
      rename(newName) {
        this.ws.send(JSON.stringify({Type: "name", Data: newName}))
      },
      ready() {
        this.ws.send(JSON.stringify({Type: "ready"}))
      }
    }
  }
</script>

<style>

</style>
