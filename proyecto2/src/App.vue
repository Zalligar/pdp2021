<template>
  <div>
    <h1>Registro de libros</h1>
    <form @submit.prevent="estatusEditar ? actualizar() : crearLibro()">
      <p>
        <input type="text" v-model="titulo" placeholder="Título">
        <input type="text" v-model="autor" placeholder="Autor">
        <button type="submit">{{estatusEditar ? "Actualizar" : "Agregar"}}</button>
        <button v-if="estatusEditar" @click="cancelaEdicion()">Cancelar</button>
      </p>
    </form>

    <br>
    <img src="https://cdn2.scratch.mit.edu/get_image/gallery/289341_170x100.png" v-if="cargando">
    <br>

    <table>
      <thead>
        <tr>
          <th>Título</th>
          <th>Autor</th>
        </tr>
      </thead>
      <tr v-for="libro in libros" :key="libro.titulo">
        <td>{{libro.titulo}}</td>
        <td>{{libro.autor}}</td>
        <button @click="borrarLibro(libro)">Eliminar</button>
        <button @click="seleccionar(libro)">Editar</button>
      </tr>
    </table>
  </div>
</template>

<script>
import {db} from './firebase'

export default {
  name: 'App',
  data(){
    return {
      libros:[],
      id: "",
      titulo: "",
      autor: "",
      cargando: false,
      estatusEditar: false,
    }
  },
  created(){
    this.listarLibros();
  },
  methods:{
    async listarLibros(){
      this.cargando = true;
      const data = await db.collection("libros").get();
      this.libros = data.docs.map(doc => ({
        id: doc.id, ...doc.data()
      }))
      this.cargando = false;
      console.log(this.libros)
    },
    async crearLibro(){
      await db.collection('libros').add({
        titulo: this.titulo,
        autor: this.autor,
      })
      this.titulo = "";
      this.autor = "";
      this.listarLibros();
    },
    async borrarLibro(libro){
      if (confirm(`¿Estás seguro que desear eliminar el libro ${libro.titulo}?`)){
        await db.collection('libros').doc(libro.id).delete();
        this.listarLibros();
      }
    },
    seleccionar(libro){
      this.estatusEditar = true;
      this.id = libro.id;
      this.titulo = libro.titulo;
      this.autor = libro.autor;
    },
    async actualizar(){
      await db.collection('libros').doc(this.id).set({
        titulo: this.titulo,
        autor: this.autor
      })
      this.estatusEditar = false;
      this.id = ""
      this.titulo = ""
      this.autor = ""
      this.listarLibros();
    },
    cancelaEdicion(){
      this.estatusEditar = false;
      this.id = ""
      this.titulo = ""
      this.autor = ""
    }
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 10px;
}
body{background-color: #19b698}
th{color: #000000}
table{
  width: 95%;
  class: center;
}
</style>
