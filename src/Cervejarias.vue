<template>  

<div class="container">

  <a class="fixo button is-large is-danger is-loading" v-show="isLoading">Loading</a>
  
  
    <h1 class="title">{{title}}</h1>
    <div class="columns">
      <div class="column is-5">
        <p class="control has-addons">
          <input class="input is-expanded" type="text" placeholder="Procure pelo nome" v-model="search">
        </p>

        <p>
        </br>
         <a class="button is-info" @click.prevent="newBreweries">Novo</a>
        </p>
        

      </div>
      <div class="column is-5">
        <a class="button is-info" @click="searchBreweries">Search</a>
      </div>
    </div>
    <div class="columns">
      <div class="column is-12">
        <table class="table is-narrow is-bordered">
          <thead>
            <th>Nome</th>
            <th>Cidade</th>
            <th>Telefone</th>
            <th>Mais</th>
            <th>Ações</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="brewery of breweries">
            <td>{{brewery.name}}</td>
            <td>{{brewery.city}}</td>
            <td>{{brewery.phone}}</td>
            <td class="is-icon">
              <a href="#">
                <i class="fa fa-map-marker"></i>
              </a>
              <a href="#">
                <i class="fa fa-plus-circle"></i>
              </a>
            </td>
            <td class="is-icon">

              <a href="#" @click.prevent="editBrewery(brewery)">  
                <i class="fa fa-edit"></i>
              </a>
              <a href="#" @click.prevent="removeBrewery(brewery)">
                <i class="fa fa-trash"></i>
              </a>
            </td>
          </tr>
        </tbody>
      </table>
         <Pagination :total="total" :page="page" :itens-per-page="itensPerPage" @change-page="onChangePage"></Pagination>
  </div>
  
</div> 

<div class="modal" :class="{'is-active':showModal}">  
  <div class="modal-background"></div>
  <div class="modal-card">
    <header class="modal-card-head">
      <p class="modal-card-title">Cervejaria: {{selected.name}}</p>
      <button class="delete" @click.prevent="showModal=false"></button>
    </header>
    <section class="modal-card-body">

    <div class="columns">
      <div class="column">
        <label class="label">Nome</label>
          <p class="control">
            <input class="input" type="text" placeholder="Text input" v-model="selected.name">
          </p>
      </div>
      <div class="column">
         <label class="label">Código</label>
    <p class="control">
      <input class="input" type="text" placeholder="Código" v-model="selected.code">
    </p>
      </div>
      </div>

      <label class="label">Descrição</label>
      <p class="control">
        <textarea class="textarea" placeholder="Textarea" v-model="selected.descript"></textarea>
      </p>

   <label class="label">Website</label>
    <p class="control">
      <input class="input" type="text" placeholder="Text input" v-model="selected.website">
    </p>


    </section>
    <footer class="modal-card-foot">
      <a class="button is-primary" @click.prevent="saveBrewery">Salvar</a>
      <a class="button" @click.prevent="showModal=false">Cancelar</a>
    </footer>
  </div>
</div>  



</template>  
<script>

import Pagination from './Pagination.vue'

export default {
  data() {
    return {
      isLoading: false,
      title: "Vue.js Crud",
      search: "",
      breweries: [],
      page: 1,
      total: 0,
      selected: {},
      itensPerPage: 10,
      showModal:false
    };
  },

  components: {
      Pagination
    },

  methods: {
    
    onChangePage(page) {
    this.page = page;
    this.loadBreweries();
    },

    showLoading() {
      this.isLoading = true;
    },
    hideLoading() {
      this.isLoading = false;
    },

    loadBreweries() {
      let t = this;
      this.showLoading();

      let start = (this.page * this.itensPerPage) - (this.itensPerPage);
      let end = this.page * this.itensPerPage;
      let qString = "";

       if (this.search){
          qString = `&q=${this.search}`
        }

      this.$http
        .get(`/breweries?_start=${start}&_end=${end}${qString}`)
        .then(
          response => {
            t.breweries = response.body;
            t.total = response.headers.get('X-Total-Count');
            console.log(t.total);
          },
          error => {
            console.log(error);
          }
        )
        .finally(function() {
          t.hideLoading();
        });
    },

    searchBreweries() {
        this.loadBreweries();
    },

    editBrewery(brewery){
        this.selected=brewery
        this.showModal = true;
       },

saveBrewery(){
    console.log(this.selected.id);                                                                                
        if (this.selected.id!=null){  //EDIT
          this.$http.put(`/breweries/${this.selected.id}`,this.selected).then(
            response=>{
              this.$set('selected',{})
              this.$set('showModal',false)
            },
            error=>{
              console.error(error)
            }
            ).finally(function() {
              this.loadBreweries();
            })
          }
          else
          { //NEW
            this.$http.post(`/breweries`,this.selected).then(
            response=>{
              this.$set('selected',{})
              this.$set('showModal',false)
            },
            error=>{
              console.error(error)
            }
            ).finally(function() {
              this.loadBreweries();
            })
          }
},

 newBreweries(){
        this.selected={}
        this.showModal = true;
       },

removeBrewery(brewery){  
        let self = this;
        swal({  title: "Você tem certeza?",
                 text: `Deseja apagar "${brewery.name}"`,   
                 type: "warning",   
                 showCancelButton: true,   
                 confirmButtonColor: "#DD6B55",   
                 cancelButtonText: "Cancelar",
                 confirmButtonText: "Sim, pode apagar!", 
                 showLoaderOnConfirm: true,  
                 closeOnConfirm: false }, function(){   

                  self.$http.delete(`/breweries/${brewery.id}`).then(
                    result=>{
                      swal("Cervejaria removida!")
                      self.loadBreweries()
                    })
        });
       },
  },
   

  created() {
    this.loadBreweries();
  }

};
</script>  
<style>
.fixo {
  float: right;
  margin-right: 10px;
  margin-top: 0px;
  z-index: 1000;
}
</style>  