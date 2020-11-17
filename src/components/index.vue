<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
        <v-img
          :src="require('../assets/logo.svg')"
          class="my-3"
          contain
          height="200"
        />
        
      </v-col>


      <v-col class="mb-4">
        <h1 class="display-2 font-weight-bold mb-3">
          Welcome to demo chat
        </h1>

        <p class="subheading font-weight-regular">
          For help and understanding how it work,
          <br>please read
          <a
            href="https://community.vuetifyjs.com"
            target="_blank"
          >documentation</a>
        </p>
      </v-col>

      <v-col
        class="mb-5 text-left"
        cols="12"
      >
  
        <v-card
          max-width="450"
          class="mx-auto"
        >
          <v-toolbar color="pink" dark>
            <v-toolbar-title>My chat</v-toolbar-title>
            <v-spacer></v-spacer>
            <v-btn icon @click="toggleSubscribe()">
              <v-icon>{{subscribe_id?'mdi-message-bulleted':'mdi-message-bulleted-off'}}</v-icon>
            </v-btn>
          </v-toolbar>

          <v-list two-line height="400" class="scroll" id="msgList">
            <template v-for="(item, index) in items">
              <v-list-item
                :key="index"
              >
                <v-list-item-avatar v-if="!item.itIsMine">
                  <v-img :src="item.avatar"></v-img>
                </v-list-item-avatar>

                <v-list-item-content :class="item.itIsMine?'text-right':'text-left'">
                  <v-list-item-title v-html="item.title"></v-list-item-title>
                </v-list-item-content>

                <v-list-item-avatar v-if="item.itIsMine">
                  <v-img :src="item.avatar"></v-img>
                </v-list-item-avatar>

              </v-list-item>
            </template>
          </v-list>
          <v-divider></v-divider>
          <v-card-actions>
            <v-text-field
              v-model="message"
              hide-details
              outlined
              dense
              placeholder="Enter message"
              @keydown.enter="publish"
            ></v-text-field>
            <v-btn
              color="pink"
              dark
              class="ml-2"
              @click="publish"
            >
              <v-icon>mdi-send-outline</v-icon>
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  export default {
    data: () => ({
      event_name:'new_msg',
      subscribe_id:null,
      message:null,
      items: [],
      avatar:null,
      avatars:[
        'https://cdn.vuetifyjs.com/images/lists/1.jpg',
        'https://cdn.vuetifyjs.com/images/lists/2.jpg',
        'https://cdn.vuetifyjs.com/images/lists/3.jpg',
        'https://cdn.vuetifyjs.com/images/lists/4.jpg',
        'https://cdn.vuetifyjs.com/images/lists/5.jpg',
      ]
    }),
    methods:{
      toggleSubscribe:function(){
        if(this.subscribe_id){
          this.unsubscribe()
        }else{
          this.subscribe()
        }
      },
      subscribe:function(){
        
        var self=this;

        window.EventTube.sub(this.event_name,function(msg){
          
          msg.data=JSON.parse(msg.data)
          
          //Обработчик события
          self.items.push({
            itIsMine:(window.EventTube.ws.clientId==msg.data.owner),
            avatar: msg.data.avatar,
            title: msg.data.text,
          });

          self.$nextTick(() => {
            var list=document.getElementById("msgList")
            list.scrollTo({
              top: list.scrollHeight,
              left: 0,
              behavior: 'smooth'
            });
          });

        }).then(function(subId){
          // Обработчик успешной подписки
          self.subscribe_id=subId;
        },function(err){
          // Обработчик ошибки подписки
          console.log('Sub err:',err);
          self.subscribe_id=null;
        });
      },
      unsubscribe:function(){
        var self=this;
        window.EventTube.unSub(this.event_name).then(function(data){
          self.subscribe_id=null;
          console.log(data);
        },function(err){
          console.log('Unsub err:',err);
        });
      },
      publish:function(){
        if(this.message==''){
          return
        }
        var msg={
          owner:window.EventTube.ws.clientId,
          avatar:this.avatar,
          text:this.message,
        }
        window.EventTube.pub(this.event_name,JSON.stringify(msg));
        this.message='';
      },
    },
    mounted:function(){
      var self=this;

      var avatarIndex=Math.floor(Math.random() * Math.floor(self.avatars.length));
      self.avatar=self.avatars[avatarIndex];
      
      // дожидаюсь зарузки SDK
      setTimeout(() => {
        self.subscribe()
      }, 200);
      
    }
  }
</script>
<style scoped>
  .scroll {
    overflow-y: scroll
  }
</style>