<template>
  <div>
    <h2>Chat Room Page</h2>    
    <form @submit.prevent="sendMessage">
        <input type="text" placeholder="Message..." v-model="message" />
        <input type="submit" value="Send" />
    </form>
  </div>
</template>

<script>
export default {
    data() {
        return {
            message: ''
        };
    },
    methods: {
        sendMessage: function() {
            socket.emit('message', this.message); 
            this.message = '';
        }
    }

}
</script>
 <style scoped>
 
 </style>