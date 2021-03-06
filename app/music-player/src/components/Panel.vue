<template>
  <div
    class="panel"
    :style="{ 'background': backgroundColor }">
    <img
      v-if="currentCover"
      class="cover"
      :src="currentCover"/>
    <div
      class="info"
      :style="{ 'color': foregroundColor }">
      <div>
        {{ name }}
      </div>
      <div>
        {{ artist }}
      </div>
    </div>
    <div
      class="control"
      :style="{ 'color': foregroundColor }">
      <div class="current-time">
        {{ currentTime }}
      </div>
      <font-awesome-icon
        class="backward"
        :style="{ 'color': foregroundColor }"
        icon="step-backward"
        @click="playPrevItem"
      />
      <font-awesome-icon
        class="play"
        :style="{ 'color': foregroundColor }"
        :icon="playIcon"
        @click="toggle"
      />
      <font-awesome-icon
        class="forward"
        :style="{ 'color': foregroundColor }"
        icon="step-forward"
        @click="playNextItem"
      />
      <div class="duration">
        {{ duration }}
      </div>
    </div>
    <div class="visual">
      <audio ref="player">
        <source :src="currentTrack">
      </audio>
      <av-bars
        class="visual-bar"
        ref-link="player"
        caps-color="#FFF"
        :bar-color="['#f00', '#ff0', '#0f0']"
        :caps-height="2"
      />
    </div>
  </div>
</template>

<script>
 import { mapState } from "vuex";
 import albumArt from "album-art"

 export default {
   name: 'Panel',
   data() {
     return {
       currentTime: "",
       currentCover: "",
       duration: "",
       name: "",
       artist: "",
       backgroundColor: "",
       foregroundColor: "",
       playIcon: "play-circle"
     }
   },
   computed: mapState([
     "currentTrack",
     "currentTrackIndex",
     "fileInfos"
   ]),
   watch: {
     "fileInfos": function() {
       this.playItem(this.fileInfos[0]);
     }
   },
   props: {
   },
   mounted() {
     window.initPanelColor = this.initPanelColor;
     window.forward = this.forward;
     window.backward = this.backward;
     window.toggle = this.toggle;
     window.playNextItem = this.playNextItem;
     window.playPrevItem = this.playPrevItem;

     this.$root.$on("playItem", this.playItem);

     let that = this;
     this.$refs.player.addEventListener("ended", function(){
       that.playNextItem();
     });

     this.$refs.player.addEventListener('timeupdate', () => {
       that.currentTime = that.formatTime(that.$refs.player.currentTime);
       that.duration = that.formatTime(that.$refs.player.duration);
     });
   },
   methods: {
     playItem(item) {
       this.$store.commit("updateCurrentTrack", item.path);

       this.playIcon = "pause-circle";

       this.name = item.name;
       this.artist = item.artist;

       this.$refs.player.load();
       this.$refs.player.play();

       albumArt(item.artist, (error, response) => {
         if (error) {
           this.currentCover = "";
         } else {
           this.currentCover = response;
         }
       })
     },

     initPanelColor(backgroundColor, foregroundColor) {
       this.backgroundColor = backgroundColor;
       this.foregroundColor = foregroundColor;
     },

     formatTime(seconds) {
       if (isNaN(seconds)) {
         return "00:00"
       } else {
         var minutes = Math.floor(seconds / 60);
         minutes = (minutes >= 10) ? minutes : "0" + minutes;
         seconds = Math.floor(seconds % 60);
         seconds = (seconds >= 10) ? seconds : "0" + seconds;
         return minutes + ":" + seconds;
       }
     },

     forward() {
       this.$refs.player.currentTime += 10;
     },

     backward() {
       this.$refs.player.currentTime -= 10;
     },

     toggle() {
       if (this.$refs.player.paused) {
         this.$refs.player.play();
         this.playIcon = "pause-circle";
       } else {
         this.$refs.player.pause();
         this.playIcon = "play-circle";
       }
     },

     playPrevItem() {
       var currentTrackIndex = this.currentTrackIndex;

       if (currentTrackIndex > 0) {
         currentTrackIndex -= 1;
       } else {
         currentTrackIndex = this.fileInfos.length - 1;
       }

       this.playItem(this.fileInfos[currentTrackIndex]);
     },

     playNextItem() {
       var currentTrackIndex = this.currentTrackIndex;

       if (currentTrackIndex < this.fileInfos.length - 1) {
         currentTrackIndex += 1;
       } else {
         currentTrackIndex = 0;
       }

       this.playItem(this.fileInfos[currentTrackIndex]);
     },
   }
 }
</script>

<style scoped>
 .panel {
   position: absolute;
   bottom: 0;

   width: 100%;

   box-shadow: 0px -4px 3px rgba(30, 30, 30, 0.75);

   display: flex;
   flex-direction: row;
   align-items: center;
 }

 .cover {
   height: 80%;
   margin-left: 30px;
 }

 .info {
   width: 30%;
   user-select: none;
   padding-left: 30px;

   overflow: hidden;
   white-space: nowrap;
   text-overflow: ellipsis;
 }

 .control {
   display: flex;
   flex-direction: row;
   align-items: center;
   width: 40%;
   justify-content: center;
 }

 .visual {
   width: 30%;
   padding-right: 30px;
 }

 .visual-bar {
   display: flex;
   justify-content: flex-end;
 }

 .backward {
   font-size: 24px;
   cursor: pointer;
 }

 .forward {
   font-size: 24px;
   cursor: pointer;
 }

 .play {
   font-size: 40px;
   margin-left: 10px;
   margin-right: 10px;
   cursor: pointer;
 }

 .current-time {
   margin-right: 20px;
 }

 .duration {
   margin-left: 20px;
 }
</style>
