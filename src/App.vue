<script setup>
import vmodal from 'vue-js-modal'
import { ref } from 'vue'
import { initializeApp } from "firebase/app";
import { getFirestore, collection, doc, setDoc, getDoc, getDocs, deleteDoc } from "firebase/firestore";

const firebaseConfig = {
  apiKey: "AIzaSyCc2EsVV_j7L4vLqn1iWobWvpzGG7FbUpI",
  authDomain: "videomanager-385522.firebaseapp.com",
  projectId: "videomanager-385522",
  storageBucket: "videomanager-385522.appspot.com",
  messagingSenderId: "525603260252",
  appId: "1:525603260252:web:8a361ee6e04bfcb4eee399"
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);
var videoEnBD = ref(false)
const urlInput = ref('')
var tituloModalMsg = ref('')
var msgModalMsg = ref('')
var listaVideos = ref([])
var showModalVideo = ref(false)
var showModalVideoExiste = ref(false)
var videoObject = ref({
  
})



async function fetchData() {
  todoData.value = null
  const res = await fetch(
    `https://jsonplaceholder.typicode.com/todos/${todoId.value}`
  )
  todoData.value = await res.json()
}

async function agregarVideo(id,titulo,duracion,img,url,descripcion){
  await setDoc(doc(db, "videos", id), {
      
      titulo: titulo,
      duracion: duracion,
      img: img,
      url: url,
      descripcion: descripcion,
      id: id
    }

    );

  var video = await getDoc(doc(db, "videos", id));
  if (video.exists()) {
    console.log("Document data:", video.data());
    showModalVideoExiste.value = true
    tituloModalMsg.value = 'Video agregado correctamente ✅'
    msgModalMsg.value = 'El video se agrego correctamente a la base de datos'
  } else {
    console.log("No such document!");
  }

}

async function cargarVideos(){
  const videos = await getDocs(collection(db, "videos"));
  listaVideos.value = []
  videos.forEach((doc) => {
    
    listaVideos.value.push(doc.data())
  });
  console.log(listaVideos.value)

}

function comprobarVideoExiste(id){
  var existe = false
  listaVideos.value.forEach(video => {
    if(video.id == id){
      existe = true
    }
  });
  return existe
}

async function obtenerVideo(id, urlVideo){
  var videos =  await fetch('https://youtube.googleapis.com/youtube/v3/videos?part=contentDetails&part=snippet&id='+id+'&key=AIzaSyCc2EsVV_j7L4vLqn1iWobWvpzGG7FbUpI')
  videos = await videos.json()
  videos = videos.items[0]
  if(videos == undefined){
    console.log('el video no existe')
    showModalVideoExiste.value = true
    tituloModalMsg.value = 'Error al ingresar video ❌'
    msgModalMsg.value = 'El video no existe en youtube'
    
  }
  console.log(videos.snippet)
  
  
    await agregarVideo(id,videos.snippet.title,videos.contentDetails.duration,videos.snippet.thumbnails.high,urlVideo,videos.snippet.description)
    cargarVideos()
  
}

async function obtenerVideoFromInput(){
  var idVideo = ''
  console.log(urlInput.value)
  var idVideo = urlInput.value.split('v=')[1]
  console.log(idVideo)
  if (idVideo == undefined){
    idVideo = urlInput.value.split('be/')[1]
  }
  if (idVideo == undefined){
    showModalVideoExiste.value = true
    tituloModalMsg.value = 'Error al ingresar video ❌'
    msgModalMsg.value = 'El link no corresponde a un enlace a youtube'
  }else{
    var ampersandPosition = idVideo.indexOf('&');
    if(ampersandPosition != -1) {
      idVideo = idVideo.substring(0, ampersandPosition);
    }
    console.log(idVideo)
    var videoExiste = comprobarVideoExiste(idVideo)
    if (videoExiste == false){
      if(listaVideos.lenght < 12){
        showModalVideoExiste = false
        tituloModalMsg.value = 'Error al ingresar video ❌'
        msgModalMsg.value = 'La base de datos esta llena'
      }
      await obtenerVideo(idVideo,urlInput.value)
    }
    else{
      console.log('el video ya existe')
      showModalVideoExiste.value = true
      tituloModalMsg.value = 'Error al ingresar video ❌'
      msgModalMsg.value = 'El video ya existe en la base de datos'
    }
    }
  
}

async function eliminarVideo(id){
  
  await deleteDoc(doc(db, "videos", id));
  cargarVideos()
}

function obtenerDuracion(duracion){
  var duracion = duracion.split('PT')[1]
  duracion = duracion.split('M')
  var minutos = duracion[0]
  var segundos = duracion[1].split('S')[0]
  return minutos+':'+segundos
}

function mostrarModal(video){
  videoObject.value = video
  showModalVideo.value = true
}

function cerrarModal(tipo){
  if(tipo=='msg'){
    showModalVideoExiste.value = false
  }else{
    showModalVideo.value = false
  }
  
}

function redireccionar(url){
  window.open(url, '_blank').focus();
}

cargarVideos()
</script>

<template>
  <div>
    <div class="body">
      <h1>Video Manager App</h1>
      <div class="buscador">
        <input v-on:input="urlInput = $event.target.value" placeholder="Ingrese URL de youtube..."   />
        <button v-on:click="obtenerVideoFromInput">Añadir</button>
      </div>
      
      <p v-if="!listaVideos">Cargando videos...</p>
      <pre v-else>
        <div class="tableroVideos">
          
          <div  class="video" v-for="video in listaVideos">
            <img v-on:click="mostrarModal(video)" :src="video.img.url" alt="" />
            <button v-on:click="eliminarVideo(video.id)">X</button>
            <div class="duracion">
              <a>{{ obtenerDuracion(video.duracion) }}</a>
            </div>
            
   
          </div>
        </div>
      </pre>
    </div>
    <div  v-if="showModalVideo" class="modal">
      <div class="modal-content">
        <button class="close-modal" v-on:click="cerrarModal">x</button>
        <div class="row-modal-content">
          <img :src="videoObject.img.url" alt="" />
          <div class="info-vid">
            <div class="titulo-video"><a>{{ videoObject.titulo }}</a></div>
            <div class="descripcion-video"><a>{{videoObject.descripcion}}</a></div>
            <button v-on:click="redireccionar(videoObject.url)">Ver video</button>
          </div>
        </div>
      </div>
    </div>
    <div  v-if="showModalVideoExiste" class="modal">
      <div class="modal-content-msg">
        <button class="close-modal-msg" v-on:click="cerrarModal('msg')">x</button>
        <div class="titulo-modal">
          <a class="titulo-modal-text">{{ tituloModalMsg }}</a>
        </div>
        <div class="mensaje-modal">
          <a class="mensaje-modal-text">{{ msgModalMsg }}</a>
        </div>
        <div class="btn-msg"><button v-on:click="cerrarModal('msg')" class="btn-aceptar-msg">Aceptar</button>
        
        
      </div>
        </div>
      </div>
    </div>

</template>

<style scoped>

.btn-msg{
  display: flex;
  justify-content: center;
  align-items: center;
}
.btn-aceptar-msg{
  height: 3vh;
  width:  10vw;
  background: linear-gradient(to right, #00cc99 0%, #3366ff 100%);
  color: #FFFFFF;
  border: none;
  border-radius: 20px;
  font-size: 1.5vh;
  margin-top: 2vh;
  margin-bottom: 2vh;
  margin-left: 2vw;
  margin-right: 2vw;
  display: flex;
  justify-content: center;
  align-items: center;
}

.btn-aceptar-msg:hover{
  background: linear-gradient(to right, #3366ff 0%, #00cc99 100%);
  cursor: pointer;
}
.mensaje-modal-text{
  font-size: 2vh;
  color: #000000;
  
}
.mensaje-modal{
  width: 100%;
  height: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.titulo-modal{
  width: 100%;
  height: 10%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.titulo-modal-text{
  font-size: 3vh;
  color: #000000;
  font-weight:  bold;
}
.close-modal{
  width:  2vw;
  background: linear-gradient( 135deg, #FCCF31 10%, #F55555 100%);
  color: #FFFFFF;
  border: none;
  border-radius: 20px;
  font-size: 2vh;

}

.close-modal:hover{
  background: linear-gradient( 135deg, #F55555 10%, #FCCF31 100%);
  cursor: pointer;
}

.close-modal-msg{
  width:  2vw;
  background: linear-gradient(to right, #00cc99 0%, #3366ff 100%);

  color: #FFFFFF;
  border: none;
  border-radius: 20px;
  font-size: 2vh;

}

.close-modal-msg:hover{
  background: linear-gradient(to right, #3366ff 0%, #00cc99 100%);
  cursor: pointer;
}
.info-vid{
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 50%;
  height: 100%;
  
  padding: 20px;
}

.info-vid button{
  width: 100%;
  height: 10%;
  margin-top: 10vh;
  background-color: #FF0000;
  color: #FFFFFF;
  border: none;
  border-radius: 20px;
  font-size: 2vh;
  font-weight: bold;
  cursor: pointer;
}
.titulo-video{
  
  
  height: 10vh;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 1;
  -webkit-box-orient: vertical;
}

.titulo-video a{
  color: #000000;
  text-decoration: none;
  font-size: 2vh;
  font-weight: bold;
}

.descripcion-video{
  
  width: 100%;
  height: 100px;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
}

.descripcion-video a{
  color: #000000;
  text-decoration: none;
  font-size: 1.5vh;
  
}


.modal {

  display: flex;
  justify-content:  center;
  align-items: center;
  position: fixed; 
  z-index: 0; 
  padding-top: 100px;
  left: 0;
  top: 0;
  width: 100vw; 
  height: 100vh; 
  overflow: auto; 
  background-color: rgb(0,0,0);
  background-color: rgba(0,0,0,0.4);
}

.row-modal-content img{
  width: 45%;
  height: 85%;
  border-radius:  20px;
  box-shadow:  0 0 50px rgba(0,0,0,0.5);
}

.row-modal-content{
  display: flex;
  justify-content: space-between ;
  align-items: center;
  flex-direction: row;
  
  width: 100%;
  height: 100%;
}

.modal-content{
  z-index: 1;
  background-color: #fefefe;
  margin: auto;
  padding: 10px;
  border: 1px solid #888;
  width: 60%;
  height: 50%;
  border-radius:  20px;
  box-shadow:  0 0 50px rgba(0,0,0,0.5);
}

.modal-content-msg{
  z-index: 1;
  background-color: #fefefe;
  margin: auto;
  padding: 10px;
  border: 1px solid #888;
  width: 40%;
  height: 30%;
  border-radius:  20px;
  box-shadow:  0 0 50px rgba(0,0,0,0.5);
}

.body h1{
  font-size: 2rem;
  color: #00cc99;
  margin-bottom: 20px;
  margin-top: 20px;
  font-family:  'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  font-weight:  600;
}
  .body {
    display: flex;
    align-items: center;
    flex-direction: column;
    background-color: #fff;
    width: 96vw;
    height: 90vh;
    border-radius:  20px;
    box-shadow:  0 0 50px rgba(0,0,0,0.5);
  }

  .buscador{
    display: flex;
    flex-direction: row;
    width: 60%;
    height: 50px;
    border-radius:  20px;

  }

  .body input{
    width: 80%;
    height: 30px;
    border-radius:  20px 0 0 20px  ;
    border: 1px solid #00cc99;
    padding: 10px;
    margin-bottom: 20px;
  }

  .body input:focus{
    outline: none;
    border: 2px solid #00cc99;
  }

  .buscador button{
    width: 20%;
    height: 30px;
    border-radius:  0 20px 20px 0;
    border: none;
    background: linear-gradient(to right, #00cc99 0%, #3366ff 100%);
    color: #fff;
    font-weight: 600;
    cursor: pointer;
  }

  .tableroVideos{
    display: flex;
    flex-direction: row;
    width: 87vw;
    height: 70vh;
    border-radius:  20px;
    
    flex-wrap:  wrap;
  }

 

  .video{
    display: flex;
    flex-direction: column;
    width: 20vw;
    height: 20vh;
    border-radius:  20px;
    margin-left: 10px;
    margin-right: 10px;
    box-shadow:  0 0 50px rgba(0,0,0,0.5);
    


  }

  .video:hover{
    box-shadow:  0 0 50px rgba(0,0,0,0.8);
    cursor: pointer;
  }

  .video img{
    width: 100%;
    height: 100%;
    border-radius:  20px;
    

  }

  .video button{
    position:  absolute;
    bottom: 80%;
    width: 10%;
    left: 85%;
    height: 30px;
    border-radius: 20px;
    border: none;
    background: linear-gradient(to right, #00cc99 0%, #3366ff 100%);
    color: #000;
    font-weight: 600;
    cursor: pointer;
    box-shadow:  0 0 90px rgba(0,0,0,0.5);

  }

  .video button:hover{
    background: linear-gradient(to right, #3366ff 0%, #00cc99 100%);
    
  }

  .duracion{
    position:  absolute;
    bottom: 5%;
    width: 25%;
    height: 30px;
    left: 70%;
    border-radius: 20px;
    border: none;
    background: linear-gradient(to right, #00cc99 0%, #3366ff 100%);
    color: #fff;
    font-weight: 600;
    cursor: pointer;
    box-shadow:  0 0 90px rgba(0,0,0,0.5);
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .duracion:hover{
    background: linear-gradient(to right, #3366ff 0%, #00cc99 100%);
    
  }

  .duracion a{
    color: #000;
    font-family:  'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  }

  @media screen and (max-width: 1024px) {
    .body {
      width: 85vw;
    }
    
  }
</style>