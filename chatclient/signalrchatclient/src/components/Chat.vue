<template>
    <div>
      <h2>Chat</h2>
  
      <div v-for="msg in messages" :key="msg.id">
        <strong>{{ msg.user }}</strong>: {{ msg.message }}
      </div>
  
      <div>
        <input v-model="user" placeholder="User" />
        <input v-model="message" placeholder="Message" />
        <button @click="sendMessage">Send</button>
      </div>
  
      <hr />
  
      <h3>Connected Users:</h3>
      <ul>
        <li v-for="connectedUser in connectedUsers" :key="connectedUser">{{ connectedUser }}</li>
      </ul>
    </div>
  </template>
  
  <script>
  import * as signalR from "@microsoft/signalr";
  import axios from "axios";
  
  export default {
    name: "ChatVue",
    data() {
      return {
        user: "",
        message: "",
        messages: [],
        connectedUsers: [],
        hubConnection: null,
      };
    },
  
    mounted() {
      this.initializeSignalR();
      this.refreshConnectedUsers();
    },
  
    methods: {
      initializeSignalR() {
        this.hubConnection = new signalR.HubConnectionBuilder()
          .withUrl("https://localhost:5001/chatHub")
          .configureLogging(signalR.LogLevel.Information)
          .build();
  
        this.hubConnection.start().then(() => {
          console.log("SignalR connection established");
          this.registerSignalREventHandlers();
        });
      },
  
      registerSignalREventHandlers() {
        this.hubConnection.on("ReceiveMessage", (user, message) => {
          this.messages.push({ user, message });
        }); 
      }, 

      
    refreshConnectedUsers() {
        axios
        .get("https://localhost:5001/api/chat/users", {
            headers: { "Cache-Control": "no-cache" } // Add cache-control header to avoid caching
        })
        .then((response) => {
            this.connectedUsers = response.data;
        })
        .catch((error) => {
            console.error("Error fetching connected users:", error);
        });
    }, 
      sendMessage() {
        if (this.user && this.message) {
          this.hubConnection
            .invoke("SendMessage", this.user, this.message)
            .then(() => {
              console.log("Message sent");
              this.message = "";
              this.refreshConnectedUsers(); // Call refreshConnectedUsers after sending the message

            })
            .catch((error) => {
              console.error("Error sending message:", error);
            });
        }
      },
    },
  };
  </script>
  
  <style scoped>
  /* Add any required component styling here */
  </style>
  