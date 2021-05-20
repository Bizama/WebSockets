<template>
  <div>
    <h1>Chat Room Section</h1>   
    <div> 
      <ul ref="chatbox" class="chatbox-container" id="chatbox">
        <li v-for="message in messages" :key="message.name" class="list">
          {{ message.name }}({{message.date}}): {{ message.message }}
        </li>
      </ul>
      <form @submit.prevent="sendMessage" class="chatbox-bottom" v-if="this.nickname != ''">
        <input  class="chatbox-input" type="text" placeholder="Message..." v-model="message" />
        <input type="submit" value="Send" />
      </form>
      <form @submit.prevent="changeNickname" v-else>
        <span>Introduzca un nickname para chatear</span>
        <br>
        <input type="text" placeholder="Escriba su nickname..." v-model="nickname_aux" />
        <input type="submit" value="Ingresar" /> 
      </form>
    </div>
    <div class='mapbox-container'>
       <l-map 
        :min-zoom="4"
        v-model="zoom"
        v-model:zoom="zoom"
        :center="[-33.8, -70.803203]"
        ref="myMap">
         <LTileLayer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png" />
         <l-marker
            v-for="(airport, index) in airports"
            :key="`ai-${index}`"
            :lat-lng="airport"
          ></l-marker>
         <l-marker
          v-for="(plane, index) in planes"
          :key="`pls-${index}`"
          :lat-lng="plane.position"
         >
          <l-tooltip :options="{ offset: [12, 24] }">
            {{ plane.code }}
          </l-tooltip>
         </l-marker>
         <l-control-layers />
         <LPolyline 
          v-for="(poly, index) in theoricalPath"
          :key="`p-${index}`"
          :lat-lngs="poly.latlngs"
          :color="poly.color"
         />
       </l-map>
    </div>
    <div style="margin-top:32px">
      <button @click="getFlights">Obtener informacion de vuelos</button>
      <div class="flights-info"> 
        <li v-for="flight in flights[0]" :key="flight.code">
          <flights-info v-bind:flight_list="flight" />
        </li>
      </div>
    </div>

  </div>
</template>

<script>
    import io from "socket.io-client";
    import FlightsInfo from './FlightsInfo';
    import "leaflet/dist/leaflet.css";
    import { 
        LMap,
        LTileLayer,
        LMarker,
        LControlLayers,
        LTooltip,
        LPolyline,
    } from "@vue-leaflet/vue-leaflet";
    import L from 'leaflet';

    export default {
        components: {
          LMap,
          LTileLayer,
          LMarker,
          LPolyline,
          LTooltip,
          FlightsInfo,
        },
        name: 'HelloWorld',
        data() {
          return {
            socket: {}, //how to initialize in Vue
            message: '',
            messages: [],
            nickname: '',
            flights: [],
            planes: [],
          }  
        }, 
        created() {
          this.socket = io("ws://tarea-3-websocket.2021-1.tallerdeintegracion.cl", {path: '/flights'});
        },
        mounted() {
          // this.context = this.$refs.game.getContext("2d");
          this.socket.on('POSITION', data => {
            this.planes = this.planes.map((p) => {
              if (data.code === p.code) {
                p.position = data.position;
              }
              return p;
            });
          });
          this.socket.on('CHAT', data => {
            const message_date = new Date(data.date);
            data.date = message_date;
            this.messages.push(data);
            this.$nextTick(function() {
              var messageBox = this.$refs['chatbox'];
              messageBox.scrollTop = messageBox.scrollHeight;
            }); 
            }
          );
          this.socket.once('FLIGHTS', data => {
            this.flights.push(data);

            this.planes = this.flights[0].map((f) => {
              return {
                code: f.code,
                position: [...f.origin],
                destination: [...f.destination],
                icon: L.icon({
                  iconUrl: `../src/assets/logo.png`,
                  iconSize: [64, 64],
                  iconAnchor: [32, 32],
                })
              };
            })
            console.log(this.planes);
          });
        },
        methods: {
          sendMessage: function () {
            this.socket.emit('CHAT', {name: this.nickname, message: this.message} );
            this.message = ''
          },
          checkNickname: function() { 
            if (this.nickname == '') {
              return false;
            }
          },
          getFlights: function() { 
            this.socket.emit('FLIGHTS');
          },
          changeNickname: function() {
            this.nickname = this.nickname_aux 
          }
        },
        computed: {
          airports: function() {
            return new Set(
              this.flights[0].map((f) => {
                const coords = [...f.origin];
                const coords2 = [...f.destination];
                return [coords, coords2];
              }).flat()
            );
          },
          theoricalPath: function() {
            return this.flights[0].map((f)=> {
              return {
                latlngs: [[...f.origin], [...f.destination]],
                color: '#14B8A6'
              };
            });
          },
        },
        
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.chatbox-container{
    height: 50px;
    width: 400px;
    overflow-y: scroll;
  }

.chatbox-input {
  margin-left:16px;
}

.chatbox-bottom {
  width: 400px;
  justify-content: space-between;
}

.flights-info {
  height: 200px;
  width: 100%;
  padding-right: 50px;
  padding-left: 50px;
  display: flex;
  flex-direction: row;
  margin-right: 16px;
  list-style: none;
  margin-top: 32px;
}

.list {
  list-style: none;
}

.mapbox-container {
  height: 400px;
  width: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 32px;
}
</style>
