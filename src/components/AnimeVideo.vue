<template>
  <div id="container" style="margin-top: -10px; padding-top: 0;">
    <span class="pbox"></span>
    <div id="reproductor-box">
      <div class="video-header">
        <h1></h1>
        <form class="video-section">
          <select class="container" v-model="eps">
            <option v-for="(episode , index) in Array.from({length: totalEps}, (v , k) => k + 1)"
              :value="episode"
              :key="index"
            >
              {{ episode }}
            </option>
          </select>
            
        </form>              
      </div>
       <div class="embed-responsive embed-responsive-21by9">
          <iframe ref="video" style="background-color:black;" class="embed-responsive-item" id="_video" :src="videos.video"></iframe>
      </div>
    </div>   

     <div class="jumbotron jumbotron-fluid" style="background:#144463 !important">
      <div class="container">
        <h1 class="display-4">
          {{title}}<br> 
          <span style="font-size:30px" class="badge badge-secondary">{{content}}</span>     
          <span style="font-size:30px" class="badge badge-secondary">{{state}}</span>
        </h1> 
        <p class="lead">{{synopsis}}</p>
      </div>
    </div>

  </div>
</template>


<script>
  import {value , watch} from 'vue-function-api';
  import {useState , useStore , useRouter} from '@u3u/vue-hooks';

  export default{
    name: "AnimeVideo",
    setup(props , context){
      const store = useStore();
      const {route} = useRouter();
      const state = {
        ...useState(['videos' , 'isLoading'])
      };
      const params = {
        types:{ 
          totalEps: value(route.value.params.type.replace(/[^0-9]/g,'') || 25),
          content: value(route.value.params.type.split('/')[0].trim() || '')
        },
        id: value(route.value.params.id),
        title: value(route.value.params.title),
        synopsis: value(route.value.params.synopsis),
        state: value(route.value.params.state)
      };
      const eps = value(null);

      watch(() => 
        eps.value , (eps) =>{
          store.value.dispatch("GET_ANIME_VIDEO" , {
            id: params.id.value,
            eps: eps
          });
        }
      );
      
      /****
       * It needs to be fixed.
       *****/
      watch(() =>
        state.videos.video , (video) =>{
          console.log('hi: ' , video);
          state.videos.video = video;
          let src = state.videos.video;          
          if(context.refs.video){
            this.display(src);
          }
      });
    
      const createObjectURL = (object) =>{
        return (window.URL) 
          ? window.URL.createObjectURL(object)
          : window.webkitURL.createObjectURL(object);
      };
      
      const display = async(videoStream) =>{
        var video = context.refs.video;
        console.log('context: ' , video);
        let blob = await fetch(videoStream , {mode: 'no-cors'}).then(r => r.blob());
        var videoUrl = this.createObjectURL(blob);
        video.src = videoUrl;
        video.load();
      }
      
      return{
        ...state,

        eps,
        title: params.title,
        totalEps: params.types.totalEps.value,
        synopsis: params.synopsis.value,
        content: params.types.content.value,
        state: params.state.value
      }
    }
  };
</script>
